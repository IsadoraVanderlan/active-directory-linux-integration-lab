# Lab Prático — Gestão de Identidades e Acessos com Active Directory, Windows e Linux

## 1. Visão Geral

Este laboratório implementa um ambiente corporativo multiplataforma de **Gestão de Identidades e Acessos (IAM)**, utilizando o **Active Directory** como serviço central de identidade para sistemas Windows e Linux.

O ambiente demonstra autenticação centralizada, autorização baseada em grupos, separação entre usuários comuns e privilegiados, grupos aninhados, aplicação de GPOs, integração Linux com Active Directory e gestão do ciclo de vida dos acessos.

### Escopo técnico

| Área | Implementação |
|---|---|
| Identidade centralizada | Active Directory Domain Services |
| Resolução de nomes | DNS integrado ao domínio |
| Sincronização | NTP / sincronização de horário |
| Windows | Estação ingressada no domínio |
| Políticas | Group Policy Objects (GPO) |
| Linux | Debian e Rocky Linux |
| Integração Linux/AD | `realmd` + SSSD + Kerberos |
| Autorização | Grupos de segurança |
| Privilégio administrativo | `sudo` baseado em grupos |
| Gestão de acesso | Concessão, alteração e revogação |
| Modelo de segurança | Least Privilege |

---

## 2. Arquitetura

| Host | Sistema Operacional | Função | IP |
|---|---|---|---|
| `DC01` | Windows Server | Domain Controller / AD DS / DNS | `10.10.10.2` |
| `CLI01` | Windows 11 Pro | Estação cliente do domínio | `10.10.10.10` |
| `DEBIAN01` | Debian | Servidor Linux integrado ao AD | `10.10.10.30` |
| `ROCKY01` | Rocky Linux 9.8 | Servidor Linux integrado ao AD | `10.10.10.40` |

| Parâmetro | Configuração |
|---|---|
| Domínio | `lab.local` |
| Rede interna | `10.10.10.0/24` |
| Máscara | `255.255.255.0` |
| DNS interno | `10.10.10.2` |

```text
                            lab.local
                                │
                    ┌───────────▼───────────┐
                    │         DC01          │
                    │    Windows Server     │
                    │                       │
                    │  Active Directory     │
                    │  DNS / Autenticação   │
                    │     10.10.10.2        │
                    └───────────┬───────────┘
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
             ▼                  ▼                  ▼
        ┌─────────┐        ┌──────────┐       ┌──────────┐
        │  CLI01  │        │ DEBIAN01 │       │ ROCKY01  │
        │ Win 11  │        │  Debian  │       │  Rocky   │
        │   .10   │        │   .30    │       │   .40    │
        └─────────┘        └──────────┘       └──────────┘
          Cliente             Servidor           Servidor
          Windows               Linux              Linux
```

O `DC01` centraliza as identidades e os serviços de domínio. O `CLI01` representa uma estação corporativa Windows, enquanto `DEBIAN01` e `ROCKY01` representam servidores Linux cujos acessos são controlados por identidades e grupos do Active Directory.

---

# 3. Active Directory — DC01

## 3.1. Serviços de domínio

| Implementação | Configuração |
|---|---|
| Hostname | `DC01` |
| IPv4 | `10.10.10.2/24` |
| Active Directory | AD DS |
| Domínio / Forest | `lab.local` |
| DNS | `10.10.10.2` |
| Sincronização de horário | Configurada |
| Autenticação do domínio | Kerberos |

O `DC01` foi promovido a **Domain Controller** da floresta `lab.local`, centralizando identidades, grupos, autenticação e resolução DNS do ambiente.

---

## 3.2. Estrutura organizacional

Os objetos do Active Directory foram organizados em **Organizational Units (OUs)** para separar identidades e computadores e permitir a aplicação controlada de políticas.

| Objeto | Finalidade |
|---|---|
| OU `TI` | Organização dos objetos relacionados à área de TI |
| Usuários | Identidades utilizadas nos testes de acesso |
| Grupos | Controle de autorização e privilégios |
| Computadores | Objetos ingressados no domínio |

---

## 3.3. Usuários e grupos

| Usuário | Associação | Perfil |
|---|---|---|
| `admin.lab` | `GG-IT-Admins` | Privilegiado |
| `user.lab` | `GG-IT-Users` | Comum |

| Grupo | Tipo | Finalidade |
|---|---|---|
| `GG-IT-Admins` | Segurança | Identidades administrativas |
| `GG-IT-Users` | Segurança | Identidades comuns |

A autorização foi baseada em **associação a grupos**, evitando a atribuição individual de permissões diretamente aos usuários.

---

## 3.4. Grupos aninhados

Foi implementado um cenário de grupos aninhados para separar a **função da identidade** da **permissão sobre o recurso**.

```text
Usuário
   │
   ▼
Grupo de função
   │
   ▼
Grupo de acesso
   │
   ▼
Permissão
```

A associação indireta permite que uma identidade herde o acesso concedido ao grupo de acesso através de sua participação no grupo de função.

---

# 4. Group Policy — GPO

A `GPO-IT-Baseline` foi vinculada à OU correspondente e utilizada para aplicar controles de segurança centralizados.

| Política | Finalidade |
|---|---|
| Requisitos de senha | Fortalecimento da autenticação |
| Bloqueio de conta | Proteção contra tentativas repetidas |
| Bloqueio por inatividade | Proteção de sessões não supervisionadas |

### Aplicação e validação

| Operação | Comando |
|---|---|
| Atualizar políticas | `gpupdate /force` |
| Visualizar políticas aplicadas | `gpresult /r` |
| Gerar relatório | `gpresult /h C:\gpresult.html /f` |

---

# 5. Estação Windows — CLI01

O `CLI01` foi configurado como estação Windows membro do domínio.

| Configuração | Valor |
|---|---|
| Hostname | `CLI01` |
| Sistema | Windows 11 Pro |
| IPv4 | `10.10.10.10/24` |
| DNS | `10.10.10.2` |
| Domínio | `lab.local` |

A estação utiliza o DNS interno para localizar os serviços do domínio e autentica usuários através do Active Directory.

### Validação

```powershell
whoami
ipconfig /all
gpresult /r
```

A aplicação da `GPO-IT-Baseline` foi validada diretamente no `CLI01`.

---

# 6. Gestão de Acessos Locais — Linux

Antes da utilização das identidades centralizadas do Active Directory, foram configurados usuários e grupos locais nos dois servidores Linux.

A separação de privilégios foi implementada através de grupos:

```text
linuxadmin → linux-admins → sudo
linuxuser  → linux-users  → acesso padrão
```

O privilégio administrativo foi atribuído ao **grupo**, permitindo que o nível de acesso seja controlado pela associação da identidade ao grupo correspondente.

---

# 7. Debian — DEBIAN01

## 7.1. Rede e DNS

| Configuração | Valor |
|---|---|
| Hostname | `debian01` |
| IPv4 | `10.10.10.30/24` |
| DNS | `10.10.10.2` |
| Search Domain | `lab.local` |

### Validação

```bash
ip -4 addr
ip route
ping -c 4 10.10.10.2
nslookup lab.local 10.10.10.2
```

---

## 7.2. Identidades locais

| Implementação | Objeto | Comando |
|---|---|---|
| Grupo administrativo | `linux-admins` | `sudo groupadd linux-admins` |
| Grupo comum | `linux-users` | `sudo groupadd linux-users` |
| Usuário privilegiado | `linuxadmin` | `sudo useradd -m -G linux-admins linuxadmin` |
| Usuário comum | `linuxuser` | `sudo useradd -m -G linux-users linuxuser` |

### Validação

```bash
id linuxadmin
id linuxuser
```

---

## 7.3. Privilégio administrativo local

O privilégio `sudo` foi concedido ao grupo `linux-admins`.

```text
%linux-admins ALL=(ALL:ALL) ALL
```

Dessa forma:

| Identidade | Grupo | Sudo |
|---|---|---|
| `linuxadmin` | `linux-admins` | Permitido |
| `linuxuser` | `linux-users` | Negado |

---

## 7.4. Integração com Active Directory

A integração com o domínio foi realizada utilizando `realmd`, SSSD e Kerberos.

| Operação | Comando |
|---|---|
| Descobrir domínio | `realm discover lab.local` |
| Ingressar no domínio | `sudo realm join lab.local -U Administrator` |
| Validar associação | `realm list` |
| Validar SSSD | `systemctl status sssd` |
| Consultar identidade AD | `id Administrator@lab.local` |

O servidor passou a reconhecer identidades e grupos provenientes do Active Directory.

---

# 8. Rocky Linux — ROCKY01

## 8.1. Rede e DNS

| Configuração | Valor |
|---|---|
| Hostname | `rocky01` |
| IPv4 interno | `10.10.10.40/24` |
| DNS | `10.10.10.2` |
| Search Domain | `lab.local` |
| Interface interna | `eth0` |
| Interface externa | `eth1` |

### Comunicação com o Domain Controller

```bash
ping -c 4 10.10.10.2
```

Resultado validado:

```text
4 packets transmitted
4 received
0% packet loss
```

### Resolução DNS

```bash
nslookup lab.local 10.10.10.2
```

Resultado validado:

```text
Server:  10.10.10.2
Address: 10.10.10.2#53

Name:    lab.local
Address: 10.10.10.2
```

---

## 8.2. Identidades locais

| Implementação | Objeto | Comando |
|---|---|---|
| Grupo administrativo | `linux-admins` | `sudo groupadd linux-admins` |
| Grupo comum | `linux-users` | `sudo groupadd linux-users` |
| Usuário privilegiado | `linuxadmin` | `sudo useradd -m -G linux-admins linuxadmin` |
| Usuário comum | `linuxuser` | `sudo useradd -m -G linux-users linuxuser` |

### Validação

```bash
id linuxadmin
id linuxuser
```

Estrutura implementada:

```text
linuxadmin
    └── linux-admins

linuxuser
    └── linux-users
```

---

## 8.3. Privilégio administrativo local

O privilégio `sudo` foi concedido ao grupo `linux-admins`.

```text
%linux-admins ALL=(ALL) ALL
```

| Identidade | Grupo | Operação administrativa |
|---|---|---|
| `linuxadmin` | `linux-admins` | Permitida |
| `linuxuser` | `linux-users` | Negada |

A configuração demonstra a atribuição de privilégios administrativos através da **associação a grupos**, em vez da concessão direta ao usuário.

---

## 8.4. Integração com Active Directory

| Operação | Comando |
|---|---|
| Descoberta do domínio | `realm discover lab.local` |
| Ingresso no domínio | `sudo realm join lab.local -U Administrator` |
| Validação da associação | `realm list` |
| Validação do SSSD | `systemctl status sssd` |
| Identidade do AD | `id Administrator@lab.local` |

A integração permite que o `ROCKY01` reconheça usuários e grupos provenientes do domínio `lab.local`.

---

# 9. Controle de Acesso Linux com Active Directory

Após a integração, o Active Directory passou a atuar como fonte central de identidade para os servidores Linux.

```text
                  Usuário
                     │
                     ▼
              Active Directory
                     │
              ┌──────┴──────┐
              │             │
          Identidade      Grupos
              │             │
              └──────┬──────┘
                     ▼
                    SSSD
                     │
                     ▼
              Servidor Linux
                     │
                Autorização
                     │
             ┌───────┴───────┐
             ▼               ▼
          Permitido        Negado
```

## 9.1. Reconhecimento das identidades

As identidades e associações a grupos do domínio foram validadas diretamente nos servidores Linux.

```bash
id usuario@lab.local
```

O retorno apresenta a identidade do usuário e os grupos provenientes do Active Directory reconhecidos pelo SSSD.

---

## 9.2. Controle de login por grupo

O acesso aos servidores Linux foi restringido com base na associação do usuário aos grupos autorizados do Active Directory.

```text
Usuário AD
    │
    ▼
Grupo autorizado?
    │
 ┌──┴──┐
 │     │
SIM   NÃO
 │     │
 ▼     ▼
Login  Acesso
       negado
```

A permissão de acesso é administrada através do grupo, sem necessidade de configurar individualmente cada usuário nos servidores.

---

## 9.3. Sudo baseado em grupo do AD

O privilégio administrativo nos servidores Linux também foi associado a um grupo de segurança do Active Directory.

| Perfil | Autenticação | Login | Sudo |
|---|---:|---:|---:|
| Usuário comum autorizado | Permitida | Permitido | Negado |
| Usuário administrativo | Permitida | Permitido | Permitido |
| Usuário sem grupo de acesso | Permitida no domínio | Negado no servidor | — |

Isso separa **autenticação**, **autorização de acesso** e **elevação de privilégio**.

---

# 10. Gestão do Ciclo de Vida dos Acessos

A gestão dos acessos foi centralizada através da associação das identidades aos grupos do Active Directory.

| Operação | Ação administrativa | Resultado |
|---|---|---|
| Concessão | Adicionar usuário ao grupo autorizado | Acesso concedido |
| Alteração | Alterar associação entre grupos | Privilégio atualizado |
| Revogação | Remover usuário do grupo | Acesso removido |
| Desativação | Desabilitar conta no AD | Autenticação bloqueada |

---

## 10.1. Concessão

```text
Usuário
   ↓
Adicionado ao grupo
   ↓
Permissão herdada
   ↓
Acesso concedido
```

A concessão foi realizada através da associação da identidade ao grupo responsável pelo recurso.

---

## 10.2. Revogação

```text
Usuário
   ↓
Removido do grupo
   ↓
Permissão deixa de ser herdada
   ↓
Acesso revogado
```

A revogação foi realizada removendo a associação ao grupo, sem alteração direta das permissões do usuário.

---

## 10.3. Conta desabilitada

A desabilitação da conta no Active Directory bloqueou a autenticação da identidade nos recursos integrados.

```text
Conta ativa        → Autenticação permitida
Conta desabilitada → Autenticação bloqueada
```

---

## 10.4. Usuário comum x privilegiado

| Controle | Usuário comum | Usuário privilegiado |
|---|---:|---:|
| Autenticação no domínio | Permitida | Permitida |
| Login em recurso autorizado | Permitido | Permitido |
| Operações padrão | Permitidas | Permitidas |
| `sudo` | Negado | Permitido |
| Administração do sistema | Negada | Permitida |

A diferença de privilégio é determinada pelas associações a grupos, e não por permissões configuradas diretamente na identidade.

---

## 10.5. Impacto dos grupos aninhados

O acesso também foi validado através de associação indireta:

```text
Usuário
   ↓
Grupo de função
   ↓
Grupo de acesso
   ↓
Permissão
   ↓
Recurso
```

A associação entre grupos permite administrar o acesso com base na função exercida pelo usuário.

---

# 11. Least Privilege

A estrutura de autorização foi baseada no princípio de **Least Privilege**, concedendo somente os acessos necessários para cada perfil.

| Controle | Implementação |
|---|---|
| Identidades | Centralizadas no Active Directory |
| Usuários comuns | Sem privilégio administrativo |
| Administração | Grupos administrativos dedicados |
| Permissões | Atribuídas preferencialmente a grupos |
| Concessão | Associação ao grupo correspondente |
| Alteração | Mudança de associação |
| Revogação | Remoção do grupo |
| Desativação | Bloqueio centralizado da identidade |

Modelo adotado:

```text
Identidade
    ↓
Função
    ↓
Grupo
    ↓
Permissão
    ↓
Recurso
```

A utilização de grupos reduz permissões individuais, facilita auditoria e simplifica concessão e revogação de acessos.

---

# 12. Componentes de Autenticação e Autorização

| Componente | Papel no ambiente |
|---|---|
| Active Directory | Fonte central de identidades e grupos |
| DNS | Localização do domínio e seus serviços |
| Sincronização de horário | Consistência temporal entre os sistemas |
| Kerberos | Autenticação das identidades do domínio |
| SSSD | Consulta e utilização das identidades AD no Linux |
| Grupos AD | Autorização e atribuição de privilégios |
| GPO | Aplicação centralizada de políticas no Windows |
| `sudo` | Elevação controlada de privilégio no Linux |

---

# 13. Evidências Técnicas

As evidências em vídeo foram concentradas nos controles relevantes à **Gestão de Identidades e Acessos**, evitando demonstrações extensas de instalação de sistemas e pacotes.

| Evidência | Demonstração |
|---|---|
| Active Directory | OUs, usuários e grupos |
| Grupos aninhados | Associação entre grupos e herança de acesso |
| GPO | Política aplicada ao `CLI01` |
| Debian | Reconhecimento de identidades e grupos |
| Rocky | Reconhecimento de identidades e grupos |
| Autenticação | Login utilizando usuário do domínio |
| Autorização | Acesso permitido x acesso negado |
| Privilégio | Usuário comum x privilegiado |
| Concessão | Inclusão em grupo e obtenção do acesso |
| Revogação | Remoção do grupo e perda do acesso |
| Conta desabilitada | Bloqueio centralizado da autenticação |
| Sudo via AD | Privilégio administrativo baseado em grupo |

### Demonstração final

![Demonstração Técnica do Ambiente](./Videos/demonstracao-final.gif)

---

# 14. Fluxo de Gestão de Acessos

```text
                       IDENTIDADE
                           │
                           ▼
                     AUTENTICAÇÃO
                           │
                           ▼
                   ACTIVE DIRECTORY
                           │
                    Conta habilitada?
                           │
                     ┌─────┴─────┐
                    NÃO         SIM
                     │           │
                     ▼           ▼
                   NEGADO      GRUPOS
                                 │
                                 ▼
                           AUTORIZAÇÃO
                                 │
                          Grupo autorizado?
                                 │
                          ┌──────┴──────┐
                         NÃO           SIM
                          │             │
                          ▼             ▼
                        NEGADO      ACESSO
                                         │
                                  Grupo privilegiado?
                                         │
                                   ┌─────┴─────┐
                                  NÃO         SIM
                                   │           │
                                   ▼           ▼
                              ACESSO COMUM    SUDO
```

---

# 15. Decisões de Segurança

| Decisão | Implementação |
|---|---|
| Centralização de identidade | Active Directory |
| Controle de acesso | Grupos de segurança |
| Separação de funções | Usuários comuns e privilegiados |
| Privilégio administrativo | Grupos dedicados |
| Linux | Active Directory + SSSD |
| Concessão | Inclusão em grupo |
| Revogação | Remoção do grupo |
| Desativação | Desabilitação da conta no AD |
| Políticas Windows | GPO |
| Modelo de autorização | Group-Based Access Control |
| Princípio de segurança | Least Privilege |

---

## Agradecimentos

Agradecimento especial a **Edson Bezerra**  
*Manager, LATAM Cyber Security Infrastructure Services — DXC Technology*

pela mentoria, direcionamento técnico e proposta do laboratório utilizado como base para o desenvolvimento deste projeto.