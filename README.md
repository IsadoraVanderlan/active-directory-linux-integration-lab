# Lab Prático — Active Directory, GPO e Integração Multiplataforma (Windows & Linux)

## Visão Geral

Este laboratório prático demonstra a construção e validação de uma infraestrutura corporativa integrada com gestão centralizada de identidades no **Active Directory (Windows Server)**, aplicação de **Group Policies (GPO)**, gestão de permissões locais e ingresso de servidores Linux (**Debian** e **Rocky Linux**) no domínio do Active Directory.

O objetivo deste documento é fornecer uma visão clara, sequencial e objetiva de todas as etapas executadas no ambiente.

---

## Ambiente do Laboratório

| Host | Sistema Operacional | Função no Ambiente | Endereço IP |
|---|---|---|---|
| **DC01** | Windows Server | Domain Controller / DNS (`lab.local`) | `192.168.10.10` |
| **WIN01** | Windows Client | Domain Member | `192.168.10.20` |
| **DEBIAN01** | Debian | Linux Domain Member | `192.168.10.30` |
| **ROCKY01** | Rocky Linux | Linux Domain Member | `192.168.10.40` |

**Domínio:** `lab.local`

### Arquitetura da Solução

```text
                               ┌──────────────────────────┐
                               │   DC01 (Windows Server)  │
                               │   Domain Controller / DNS│
                               │         lab.local        │
                               └────────────┬─────────────┘
                                            │
               ┌────────────────────────────┼────────────────────────────┐
               │                            │                            │
               ▼                            ▼                            ▼
   ┌───────────────────────┐    ┌───────────────────────┐    ┌───────────────────────┐
   │     WIN01 (Windows)   │    │    DEBIAN01 (Debian)  │    │ ROCKY01 (Rocky Linux) │
   │     Domain Member     │    │  Linux Domain Member  │    │  Linux Domain Member  │
   └───────────────────────┘    └───────────────────────┘    └───────────────────────┘
                                            │                            │
                                            └────────── SSSD/realmd ─────┘
```

---

## Execução Sequencial do Laboratório

### ETAPA 1: Infraestrutura Windows (Active Directory, Utilizadores, Grupos e GPO)

#### 1.1. Configuração do Active Directory
- Instalação e provisionamento das funções **AD DS** e **DNS** no servidor `DC01`.
- Promoção a Controlador de Domínio (DC) e criação do domínio **`lab.local`**.
- Validação da resolução de nomes DNS interna e registros de autoridade.

- **[Vídeo — Implementação do Active Directory](#)**  
  *Demonstrando: Instalação do AD DS, promoção a DC, criação do domínio `lab.local` e validação do DNS.*

---

#### 1.2. Ingresso do Windows no Domínio (Domain Join)
- Configuração da placa de rede da estação `WIN01` apontando o servidor DNS para `192.168.10.10`.
- Ingresso do host `WIN01` no domínio `lab.local`.
- Validação do logon utilizando contas do Active Directory.

- **[Vídeo — Windows entrando no domínio](#)**  
  *Demonstrando: Configuração do cliente, resolução DNS, ingresso no domínio e autenticação de usuário.*

---

#### 1.3. Criação de Grupos e Utilizadores no AD
- **Grupos de Segurança Criados:**
  - `GG-IT-Admins` (Administradores de TI)
  - `GG-IT-Users` (Usuários de TI)
- **Contas de Utilizador Criadas:**
  - `admin.lab` ➔ Adicionado ao grupo `GG-IT-Admins`
  - `user.lab` ➔ Adicionado ao grupo `GG-IT-Users`

- **[Vídeo — Usuários, grupos e permissões no AD](#)**  
  *Demonstrando: Criação das contas `admin.lab` e `user.lab`, criação dos grupos e associação das permissões.*

---

#### 1.4. Aplicação e Validação de Regras de GPO
- Criação e vinculação da **`GPO-IT-Baseline`** com restrições e políticas de segurança na OU/domínio.
- Execução de comandos no cliente `WIN01` para propagação e geração de relatório:
  ```cmd
  gpupdate /force
  gpresult /r
  gpresult /h C:\gpresult.html
  ```

- **[Vídeo — Implementação e validação da GPO](#)**  
  *Demonstrando: Criação da GPO, aplicação no domínio, execução do `gpupdate /force` e geração do relatório `gpresult`.*

---

### ETAPA 2: Servidores Linux (Debian e Rocky Linux — Acessos Locais e Ingresso no AD)

#### 2.1. Configuração do Debian (`DEBIAN01`)
- **Criação de Utilizadores e Grupos Locais:**
  - Grupos: `linux-admins` e `linux-users`
  - Utilizadores: `linuxadmin` e `linuxuser`
  - Associação: `linuxadmin` ➔ `linux-admins` | `linuxuser` ➔ `linux-users`
- **Controle de Elevação de Privilégios (Sudoers):**
  - Concessão de acesso `sudo` total apenas para o grupo `linux-admins`.
  - Validação executada com `sudo whoami` (resultado retornado: `root`).
- **Ingresso do Debian no Domínio Active Directory:**
  - Apontamento da resolução DNS para o IP do Domain Controller (`192.168.10.10`).
  - Instalação das dependências e ingresso no domínio via `realmd` e `SSSD`:
    ```bash
    realm discover lab.local
    realm join lab.local -U admin.lab
    ```
  - Validação de identidades do AD no Linux: `id admin.lab@lab.local`

- **[Vídeo — Configuração e Integração do Debian no AD](#)**  
  *Demonstrando: Hostname, DNS, usuários/grupos locais, teste de `sudo`, ingresso no domínio via `SSSD` e login com conta do AD.*

---

#### 2.2. Configuração do Rocky Linux (`ROCKY01`)
- **Criação de Utilizadores e Grupos Locais:**
  - Grupos: `linux-admins` e `linux-users`
  - Utilizadores: `linuxadmin` e `linuxuser`
  - Associação: `linuxadmin` ➔ `linux-admins` | `linuxuser` ➔ `linux-users`
- **Controle de Elevação de Privilégios (Sudoers):**
  - Concessão de acesso `sudo` total apenas para o grupo `linux-admins`.
  - Validação executada com `sudo whoami` (resultado retornado: `root`).
- **Ingresso do Rocky Linux no Domínio Active Directory:**
  - Apontamento da resolução DNS para o IP do Domain Controller (`192.168.10.10`).
  - Instalação das dependências e ingresso no domínio via `realmd` e `SSSD`:
    ```bash
    realm discover lab.local
    realm join lab.local -U admin.lab
    ```
  - Validação do serviço: `systemctl status sssd`

- **[Vídeo — Configuração e Integração do Rocky Linux no AD](#)**  
  *Demonstrando: Hostname, DNS, usuários/grupos locais, teste de `sudo`, ingresso no domínio via `SSSD` e login com conta do AD.*

---

## Validações e Matriz de Testes

| Item Solicitado | DC01 | WIN01 | DEBIAN01 | ROCKY01 | Status |
|---|:---:|:---:|:---:|:---:|:---:|
| Subir Active Directory / DC | 🔹 | — | — | — | **⏳ PENDENTE** |
| Adicionar Windows no Domínio | — | 🔹 | — | — | **⏳ PENDENTE** |
| Criar Usuários e Grupos no AD | 🔹 | — | — | — | **⏳ PENDENTE** |
| Aplicar Regras de GPO | — | 🔹 | — | — | **⏳ PENDENTE** |
| Criar 2 Usuários e 2 Grupos Locais | — | — | 🔹 | 🔹 | **⏳ PENDENTE** |
| Atribuir `sudo root` a um Grupo Local | — | — | 🔹 | 🔹 | **⏳ PENDENTE** |
| Adicionar Distribuições Linux no AD | — | — | 🔹 | 🔹 | **⏳ PENDENTE** |

---

## Resultado Final

Ambiente corporativo multi-sistema completamente funcional. O laboratório demonstra o domínio de infraestruturas **Windows Server (Identity & GPO)** e **Linux (Administração Local, Privilege Management e Integração de Domínio via SSSD)** em perfeita sinergia.

---

## 🤝 Agradecimentos

Agradecimento especial ao **Edson Bezerra** (_Manager, LATAM Cyber Security Infrastructure Services - DXC Technology_) pela mentoria, orientações estratégicas e incentivo na estruturação deste plano de estudos.
