# 🔐 Identity & Access Management Lab

> **Active Directory · Windows · Debian · Rocky Linux · SSSD · Kerberos · GPO · Access Management**

Laboratório prático de **Identity & Access Management (IAM)** em ambiente multiplataforma, utilizando o **Microsoft Active Directory** como serviço central de identidade e grupos como principal mecanismo de autorização.

O projeto demonstra, de forma prática, **identidade centralizada, autenticação em domínio, autorização baseada em grupos, grupos aninhados, concessão e revogação de privilégios, desabilitação de identidade e Least Privilege**.

> **Modelo central do projeto:** `Identidade → Função → Permissão → Recurso`

<br/>
<br>

---

## 📋 Proposta do Projeto

> **Requisitos que deram origem ao laboratório, contemplando Active Directory, Windows, Linux, integração com o domínio e demonstrações de Gestão de Acessos.**

<details>
<summary><b>Ver requisitos completos do projeto</b></summary>

<br>

A proposta do projeto apresentada pelo gerente **Edson Bezerra** foi construir um laboratório capaz de demonstrar os principais conceitos de **Gestão de Identidades e Acessos** em ambientes Windows e Linux.

#### 1. Active Directory / Windows

- Subir um Windows Server como Domain Controller.
- Configurar Active Directory e DNS.
- Configurar NTP / sincronização de horário.
- Subir um cliente Windows e ingressá-lo no domínio.
- Criar OUs, usuários e grupos.
- Associar usuários aos respectivos grupos.
- Criar cenário de grupos aninhados.
- Criar e aplicar GPOs.
- Demonstrar a aplicação das GPOs em um computador do domínio.

#### 2. Ambiente Linux

- Subir um servidor Debian.
- Subir um servidor Rocky Linux.
- Criar usuários e grupos locais.
- Associar usuários aos respectivos grupos.
- Configurar sudo para um dos grupos.
- Demonstrar a diferença entre usuário comum e privilegiado.

#### 3. Integração Linux + Active Directory

- Integrar Debian e Rocky Linux ao domínio.
- Demonstrar autenticação utilizando usuários do domínio.
- Demonstrar reconhecimento dos grupos do AD.
- Utilizar grupos do AD no controle de privilégios.
- Configurar sudo utilizando grupo do Active Directory.
- Demonstrar diferentes níveis de privilégio de acordo com os grupos.

#### 4. Gestão de Acessos

- Demonstrar concessão de privilégio através de associação a grupo.
- Demonstrar revogação através da remoção do grupo.
- Demonstrar o comportamento de uma conta desabilitada.
- Demonstrar usuário comum x usuário privilegiado.
- Demonstrar impacto de grupos aninhados.
- Aplicar o princípio de **Least Privilege**.

#### 5. Apresentação Técnica

Durante a apresentação devem ser explicados:

- Arquitetura implementada;
- Autenticação entre Windows, Active Directory e Linux;
- Papel de DNS, NTP e Kerberos;
- Uso de grupos no controle de acesso;
- Implementação dos privilégios administrativos;
- Ciclo de vida do acesso;
- Principais riscos de segurança e possíveis mitigações.

</details>

<br/>
<br>

---

## 🗺️ Arquitetura e Ambiente Integrado

> **Visão do ambiente utilizado para centralizar identidades no Active Directory e integrar estações Windows e servidores Linux ao domínio `lab.local`.**

![Diagrama de Arquitetura e Fluxo de Gestão de Acessos](./IAM%20Access%20Management.jpg)

<details>
<summary><b>Ver detalhes da arquitetura e do DC01</b></summary>

<br>

### 🧭 Visão Geral do Ambiente

| Host | Sistema | Função | IP |
|---|---|---|---|
| `DC01` | Windows Server | Domain Controller / AD DS / DNS / Kerberos | `10.10.10.2` |
| `CLI01` | Windows 11 Pro | Cliente do domínio / GPO | `10.10.10.10` |
| `DEBIAN01` | Debian | Servidor Linux integrado ao AD | `10.10.10.30` |
| `ROCKY01` | Rocky Linux | Servidor Linux integrado ao AD | `10.10.10.40` |

**Domínio:** `lab.local`

**Rede:** `10.10.10.0/24`

**DNS interno:** `10.10.10.2`

### 🏢 DC01 — Active Directory

O `DC01` foi configurado como **Domain Controller** do domínio:

```text
lab.local
```

Serviços utilizados:

- Active Directory Domain Services;
- DNS;
- Kerberos;
- Sincronização de horário;
- Group Policy.

O DNS interno é utilizado pelos demais hosts para localizar o domínio e seus serviços.

A sincronização de horário é importante principalmente para os mecanismos de autenticação do domínio, como o Kerberos.

</details>

<br/>
<br>

---

## 🔐 Modelo IAM e Controle de Acesso (AuthZ)

Demonstra como identidades, grupos de função e grupos de permissão foram organizados para controlar privilégios sem atribuí-los diretamente aos usuários.

<details>
<summary><b>Ver identidades, grupos, autorização e Least Privilege</b></summary>

<br>

### 👤 Identidades e Grupos

Foram utilizadas duas identidades principais do domínio:

| Usuário | Função |
|---|---|
| `admin-dc` | Usuário administrativo |
| `user-dc` | Usuário comum |

Os grupos principais são:

| Grupo | Escopo | Tipo | Função |
|---|---|---|---|
| `GG-IT-Admins` | Global | Security | Representa função administrativa |
| `GG-IT-Users` | Global | Security | Representa função de usuário comum |
| `DL-Linux-Sudo` | Domain Local | Security | Concede permissão de sudo nos servidores Linux |

### 🔐 Autorização (AuthZ) — Modelo de Acesso

A estrutura final implementada foi:

```text
admin-dc
   │
   ▼
GG-IT-Admins
   │
   ▼
DL-Linux-Sudo
   │
   ├────────► DEBIAN01 ───► sudo
   │
   └────────► ROCKY01 ────► sudo
```

Para o usuário comum:

```text
user-dc
   │
   ▼
GG-IT-Users
```

O `GG-IT-Users` representa a função do usuário comum e **não concede sudo**.

### 🧩 Separação entre Identidade, Função e Permissão

O modelo foi estruturado separando três elementos:

```text
Identidade
    │
    ▼
Função
    │
    ▼
Permissão
```

No caso do usuário administrativo:

```text
Identidade
admin-dc
    │
    ▼
Função
GG-IT-Admins
    │
    ▼
Permissão
DL-Linux-Sudo
```

A permissão administrativa não é atribuída diretamente ao usuário.

Isso permite que o acesso seja administrado através de grupos.

### 🔗 Grupos Aninhados

O laboratório implementa um cenário de **Nested Groups**:

```text
admin-dc
   │
   ▼
GG-IT-Admins
   │
   ▼
DL-Linux-Sudo
```

O `admin-dc` não precisa ser membro direto do `DL-Linux-Sudo`.

Ele recebe essa associação indiretamente porque:

```text
GG-IT-Admins
```

é membro de:

```text
DL-Linux-Sudo
```

Esse modelo facilita a administração e reduz a necessidade de conceder permissões diretamente a usuários individuais.

### 🛡️ Administração do Domínio, Domain Admins e Least Privilege

`Domain Admins` é um **grupo privilegiado nativo do Active Directory**, destinado à administração do domínio. Ele foi mantido separado da cadeia utilizada para conceder sudo nos servidores Linux.

Durante a revisão da arquitetura, o `GG-IT-Admins` foi removido de `Domain Admins`.

A cadeia de administração Linux ficou:

```text
GG-IT-Admins
       │
       ▼
DL-Linux-Sudo
       │
       ▼
sudo no Debian / Rocky
```

Enquanto a administração do domínio permanece associada ao grupo privilegiado:

```text
Domain Admins
       │
       ▼
Administração do Active Directory / domínio
```

Isso aplica o princípio de **Least Privilege** e a **separação de privilégios**:

> Um usuário que precisa administrar servidores Linux não precisa, por consequência, possuir privilégios administrativos sobre todo o domínio Active Directory.

Após essa separação, o sudo foi novamente testado no Debian e no Rocky para confirmar que a permissão continuava sendo concedida exclusivamente através do `DL-Linux-Sudo`.

</details>

<br/>
<br>

---

## 💻 Windows no Domínio

> **Demonstra o ingresso do `CLI01` no domínio e a aplicação centralizada de políticas através de GPO.**

<details>
<summary><b>Ver configuração e validações do Windows</b></summary>

<br>

### 💻 CLI01 — Windows 11

O `CLI01` representa uma estação corporativa Windows ingressada no domínio.

```text
CLI01
 │
 ├── DNS ──────────► DC01
 ├── Domínio ──────► lab.local
 └── GPO ──────────► políticas centralizadas
```

Configuração principal:

```text
IP:      10.10.10.10
DNS:     10.10.10.2
Domínio: lab.local
```

A aplicação das políticas foi validada através de:

```powershell
gpupdate /force
gpresult /r
```

O `gpresult /r` permite verificar o domínio e as políticas aplicadas ao computador/usuário.

</details>

<br/>
<br>

---

## 🐧 Integração Linux + Active Directory

Demonstra a integração do Debian e Rocky Linux ao domínio, reconhecimento de identidades e grupos, autenticação Kerberos, SSSD e concessão de sudo por grupo do Active Directory.

<details>
<summary><b>Ver integração, comandos e testes nos servidores Linux</b></summary>

<br>

### 🐧 DEBIAN01

O `DEBIAN01` representa um servidor Linux integrado ao domínio.

```text
IP:      10.10.10.30
DNS:     10.10.10.2
Domínio: lab.local
```

#### Usuários locais

Foram mantidos usuários locais para administração e testes:

```text
admin-debian
user-debian
```

Grupos locais utilizados:

```text
linux-admins
linux-users
```

A separação local segue a mesma lógica:

```text
admin-debian
     │
     ▼
linux-admins
     │
     ▼
sudo
```

```text
user-debian
     │
     ▼
linux-users
     │
     ▼
sem sudo
```

#### Integração com o Active Directory

A integração utiliza:

- `realmd`;
- `SSSD`;
- Kerberos;
- DNS do domínio.

Validações principais:

```bash
realm discover lab.local
realm list
systemctl status sssd
```

Consulta de identidade:

```bash
id admin-dc@lab.local
id user-dc@lab.local
```

Após a integração, o Debian passou a reconhecer os usuários e grupos provenientes do Active Directory.

#### Sudo através do Active Directory

A regra final de sudo utiliza o grupo:

```text
DL-Linux-Sudo
```

Exemplo da regra:

```text
%dl-linux-sudo@lab.local ALL=(ALL:ALL) ALL
```

A sintaxe foi validada com:

```bash
sudo visudo -c
```

A cadeia final é:

```text
admin-dc
   ↓
GG-IT-Admins
   ↓
DL-Linux-Sudo
   ↓
sudo
```

Validação:

```bash
id admin-dc@lab.local
sudo whoami
```

Resultado esperado:

```text
root
```

Para o usuário comum:

```bash
id user-dc@lab.local
sudo -l -U 'user-dc@lab.local'
```

Resultado:

```text
usuário sem permissão para executar sudo
```

### 🐧 ROCKY01

O `ROCKY01` representa o segundo servidor Linux do laboratório.

```text
IP:      10.10.10.40
DNS:     10.10.10.2
Domínio: lab.local
```

Também foram mantidos usuários locais:

```text
admin-rocky
user-rocky
```

e grupos locais:

```text
linux-admins
linux-users
```

O Rocky utiliza o mesmo modelo centralizado do Debian:

```text
Active Directory
       │
       ▼
      SSSD
       │
       ▼
Identidade / grupos
       │
       ▼
DL-Linux-Sudo
       │
       ▼
sudo
```

Validação:

```bash
id admin-dc@lab.local
sudo -l -U 'admin-dc@lab.local'
```

Teste real:

```bash
sudo -iu 'admin-dc@lab.local'
sudo whoami
```

Resultado:

```text
root
```

### 🔐 Autenticação (AuthN) com Kerberos

O Kerberos foi utilizado para validar a autenticação das identidades do domínio.

Exemplo:

```bash
kinit admin-dc@LAB.LOCAL
```

Visualização do ticket:

```bash
klist
```

Uma autenticação bem-sucedida confirma que:

- A identidade existe no domínio;
- A credencial foi aceita;
- O host consegue se comunicar corretamente com os serviços de autenticação do domínio.

### 🔄 SSSD e Cache

O SSSD mantém informações de identidades e grupos em cache.

Durante testes de alteração de grupos foi necessário atualizar esse cache para que novas associações fossem refletidas imediatamente.

```bash
sudo sss_cache -E
sudo systemctl restart sssd
```

Esse comportamento foi observado principalmente durante os testes de concessão e revogação.

</details>

<br/>
<br>

---

## 🔄 Ciclo de Vida da Identidade e do Acesso

> **Demonstra a diferença entre usuário comum e privilegiado e acompanha o acesso desde o estado inicial até concessão, revogação, desabilitação e reabilitação da identidade.**

<details>
<summary>🔄 <b>Ver testes de concessão, revogação e estado da conta</b></summary>

<br>

### 🔑 Testes de Gestão de Acessos

#### 1. Usuário comum x usuário privilegiado

##### Usuário comum

```text
user-dc
   ↓
GG-IT-Users
   ↓
sem sudo
```

Validação:

```bash
sudo -l -U 'user-dc@lab.local'
```

Resultado:

```text
sudo negado
```

##### Usuário privilegiado

```text
admin-dc
   ↓
GG-IT-Admins
   ↓
DL-Linux-Sudo
   ↓
sudo
```

Validação:

```bash
sudo whoami
```

Resultado:

```text
root
```

### ➕ Concessão de Privilégio

O `user-dc` iniciou como usuário comum:

```text
user-dc
   ↓
GG-IT-Users
   ↓
sem sudo
```

Para realizar o teste de concessão, ele foi adicionado temporariamente ao:

```text
GG-IT-Admins
```

A nova cadeia passou a ser:

```text
user-dc
   │
   ├── GG-IT-Users
   │
   └── GG-IT-Admins
             │
             ▼
        DL-Linux-Sudo
             │
             ▼
            sudo
```

No Linux foi possível observar:

```text
GG-IT-Users
GG-IT-Admins
DL-Linux-Sudo
```

O comando:

```bash
sudo whoami
```

retornou:

```text
root
```

Isso demonstrou que a alteração da associação no Active Directory modificou o privilégio do usuário no servidor Linux.

### ➖ Revogação de Privilégio

Após o teste de concessão, o `user-dc` foi removido do:

```text
GG-IT-Admins
```

Após atualização do cache do SSSD, os grupos:

```text
GG-IT-Admins
DL-Linux-Sudo
```

deixaram de aparecer para o usuário.

A identidade voltou ao estado:

```text
user-dc
   ↓
GG-IT-Users
   ↓
sem sudo
```

A validação:

```bash
sudo -l -U 'user-dc@lab.local'
```

confirmou que o privilégio administrativo havia sido revogado.

### 🚫 Desabilitação da Conta

Também foi testado o impacto da desabilitação centralizada de uma identidade.

No Active Directory:

```powershell
Disable-ADAccount -Identity "user-dc"
```

Validação:

```powershell
Get-ADUser "user-dc" | Select-Object Name,Enabled
```

Resultado:

```text
Enabled = False
```

No Rocky foi realizada uma nova tentativa de autenticação Kerberos:

```bash
kinit user-dc@LAB.LOCAL
```

Resultado:

```text
Client's credentials have been revoked
```

Isso demonstrou que uma identidade desabilitada no Active Directory deixa de conseguir autenticar nos sistemas integrados.

Após o teste, a conta foi reabilitada:

```powershell
Enable-ADAccount -Identity "user-dc"
```

Uma nova autenticação foi realizada com sucesso.

### 🔄 Ciclo de Vida do Acesso

O laboratório demonstra dois aspectos diferentes da gestão de acessos.

#### Gestão de privilégio

```text
Estado inicial
     │
     ▼
Concessão
     │
     ▼
Revogação
```

#### Gestão do estado da identidade

```text
Conta habilitada
       │
       ▼
Desabilitação
       │
       ▼
Autenticação bloqueada
       │
       ▼
Reabilitação
       │
       ▼
Autenticação restaurada
```

</details>

<br/>
<br>

---

## 🛡️ Controles de Segurança e Riscos

> **Reúne os princípios de segurança aplicados no laboratório, os riscos identificados, as respectivas mitigações e o status do controle de login/SSH por grupo.**

<details>
<summary><b>Ver controles IAM, riscos e mitigações</b></summary>

<br>

### 🛡️ Princípios de Segurança Aplicados

#### Least Privilege

Cada identidade recebe somente os privilégios necessários para sua função.

```text
Usuário
  ↓
Grupo de função
  ↓
Grupo de permissão
  ↓
Recurso
```

#### Permissões não atribuídas diretamente ao usuário

A administração é realizada por grupos.

Isso reduz:

- Permissões individuais;
- Inconsistências;
- Dificuldade de revogação;
- Risco de privilégios esquecidos.

#### Separação entre administração Linux e administração do domínio

```text
Linux
  ↓
DL-Linux-Sudo
```

é separado de:

```text
Domain Admins
```

Um administrador Linux não precisa automaticamente ser administrador do domínio.

### ⚠️ Riscos Identificados e Mitigações

| Risco | Mitigação |
|---|---|
| Privilégio excessivo | Aplicação de Least Privilege |
| Uso de `Domain Admins` para administração Linux | Separação através de `DL-Linux-Sudo` |
| Permissões atribuídas diretamente aos usuários | Uso de grupos |
| Dificuldade de revogar acessos | Remoção centralizada de memberships |
| Conta comprometida ou desligamento de colaborador | Desabilitação centralizada no AD |
| Informações antigas em cache | Atualização do SSSD e nova sessão |
| Falha de resolução do domínio | DNS interno apontando para o DC01 |
| Problemas de autenticação Kerberos | Sincronização de horário e DNS corretos |

Em um ambiente produtivo, o modelo poderia ser complementado com:

- MFA;
- PAM / Privileged Access Management;
- Centralização de logs;
- Auditoria de alterações de grupos;
- Processos formais de aprovação;
- Revisões periódicas de acesso;
- Políticas de expiração e recertificação de privilégios.

### 🔐 Controle de Login / SSH por Grupo

> **Status atual: etapa pendente de implementação/validação final.**

O laboratório já demonstra:

- Autenticação de usuários do domínio;
- Reconhecimento de grupos do AD;
- Controle de sudo baseado em grupo;
- Concessão e revogação de privilégio;
- Bloqueio de autenticação através da desabilitação da conta.

O controle explícito de **login/SSH permitido ou negado com base em um grupo específico do Active Directory** será validado em uma etapa própria.

Por esse motivo, o README não considera esse controle como concluído neste momento.

</details>

<br/>
<br>

---

## 🎬 Demonstração Técnica e Estado Final

> **Consolida o roteiro de apresentação, as evidências que devem ser demonstradas e o estado final dos componentes e controles implementados.**

<details>
<summary><b>Ver roteiro, validações e estado final do laboratório</b></summary>

<br>

### 🎬 Roteiro da Demonstração Técnica

A apresentação deve priorizar **evidências de Gestão de Acessos**, sem gastar tempo demonstrando instalação de sistemas operacionais.

#### Active Directory

Mostrar:

- `admin-dc`;
- `user-dc`;
- `GG-IT-Admins`;
- `GG-IT-Users`;
- `DL-Linux-Sudo`;
- Associação entre os grupos;
- `Domain Admins` separado da cadeia Linux.

#### Windows

Mostrar:

```powershell
gpresult /r
```

e evidenciar a aplicação da GPO no `CLI01`.

#### Debian / Rocky

Mostrar:

```bash
id admin-dc@lab.local
```

e evidenciar:

```text
GG-IT-Admins
DL-Linux-Sudo
```

Depois:

```bash
sudo whoami
```

Resultado:

```text
root
```

Para `user-dc`:

```bash
sudo -l -U 'user-dc@lab.local'
```

Resultado:

```text
sem permissão de sudo
```

#### Gestão de acesso

Demonstrar:

1\. estado inicial;

2\. concessão;

3\. revogação;

4\. desabilitação;

5\. reabilitação.

### ✅ Estado Final do Ambiente

```text
admin-dc
   ↓
GG-IT-Admins
   ↓
DL-Linux-Sudo
   ↓
sudo no Debian e Rocky
```

```text
user-dc
   ↓
GG-IT-Users
   ↓
sem sudo
```

```text
Domain Admins
   ↓
fora da cadeia de sudo Linux
```

Estado dos componentes:

| Componente | Estado |
|---|---|
| DC01 / Active Directory | ✅ Configurado |
| CLI01 / Domínio / GPO | ✅ Configurado |
| DEBIAN01 | ✅ Integrado |
| ROCKY01 | ✅ Integrado |
| Nested Groups | ✅ Validado |
| Sudo via `DL-Linux-Sudo` | ✅ Validado |
| `GG-IT-Admins` fora de `Domain Admins` | ✅ Validado |
| Usuário comum x privilegiado | ✅ Validado |
| Concessão de privilégio | ✅ Validado |
| Revogação de privilégio | ✅ Validado |
| Conta desabilitada | ✅ Validado |
| Reabilitação da conta | ✅ Validado |
| Controle de login / SSH por grupo | ⏳ Pendente |
| Vídeos de demonstração | ⏳ Pendente |

### 🧠 Resumo do Modelo

O princípio central do projeto é:

```text
IDENTIDADE
    ↓
FUNÇÃO
    ↓
PERMISSÃO
    ↓
RECURSO
```

Aplicado ao laboratório:

```text
admin-dc
    ↓
GG-IT-Admins
    ↓
DL-Linux-Sudo
    ↓
Debian / Rocky
    ↓
sudo
```

Esse modelo permite administrar privilégios de maneira centralizada, reduzir concessões individuais e aplicar o princípio de **Least Privilege**.

</details>

<br/>
<br>

---

## Agradecimentos

Agradecimento especial a **Edson Bezerra**

(**Manager, LATAM Cyber Security Infrastructure Services — DXC Technology**)

pela mentoria, direcionamento técnico e proposta utilizada como base para o desenvolvimento deste laboratório.

</details>
