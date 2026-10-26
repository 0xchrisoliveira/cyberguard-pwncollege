# Enumeração de Objetos via LDAP & Papel do Domain Controller (AD LDAP Enumeration)

## Briefing da Missão
* **Codinome:** Operação LDAP Recon
* **Classificação:** Confidencial / Treinamento de Infraestrutura Corporativa & Active Directory
* **Alvo:** Controlador de Domínio Primário (`DC01.corp.cyberguard.local`)
* **Objetivo:** Compreender a arquitetura mestra do Domain Controller (DC), o papel do banco central `NTDS.dit`, as portas de serviço de identidade e a sintaxe de consultas LDAP para enumeração de contas privilegiadas.

No relatório de diagnóstico do ciclo 2026.2 da Cyberguard, a **questão 06** expôs uma defasagem importante: **62,5% da equipe errou a função de um Domain Controller**, com membros confundindo seu propósito com funções elementares de rede como roteamento TCP. Este desafio restaura os alicerces de infraestrutura Windows e serviços de diretório.

---

## Teoria Aprofundada

### 1. A Centralidade do Domain Controller (DC)
Em mais de 90% das redes empresariais globais, o ecossistema corporativo gira em torno do **Active Directory Domain Services (AD DS)**. O servidor que hospeda essa função é denominado **Domain Controller (DC)**:

```
                            ┌──────────────────────────────────┐
                            │      DOMAIN CONTROLLER (DC)      │
                            ├──────────────────────────────────┤
                            │ - Banco de Dados: NTDS.dit       │
                            │ - Autenticação: Kerberos (TCP 88)│
                            │ - Diretório: LDAP (TCP 389/636)  │
                            │ - Resolução de Nomes: DNS (53)   │
                            │ - Políticas Centrais: GPOs       │
                            └────────────────┬─────────────────┘
                                             │
             ┌───────────────────────────────┼───────────────────────────────┐
             ▼                               ▼                               ▼
     [ Estações Windows ]           [ Servidores de Aplicação ]       [ Usuários e Grupos ]
     Computadores de funcionários    Bancos SQL, IIS, Intranet        Políticas de Acesso & MFA
```

### 2. O Santo Graal do Domínio: `NTDS.dit`
O arquivo `C:\Windows\NTDS\NTDS.dit` é uma base de dados construída sobre o mecanismo ESE (Extensible Storage Engine). Ele contém:
* Todos os objetos (contas de usuários, grupos, computadores, senhas).
* **Os hashes NTLM e chaves Kerberos de todos os usuários corporativos**, incluindo a conta local `Administrator` e o grupo `Domain Admins`.
* A chave da conta **`krbtgt`** (utilizada para forjar Golden Tickets).

> **Aviso Crítico:** Se um atacante extrai o arquivo `NTDS.dit` ou obtém privilégios de replicação via comando `DCSync` (`secretsdump.py`), o domínio inteiro está comprometido de forma irremediável.

### 3. Tabela de Portas e Protocolos Fundamentais do Active Directory

| Porta | Protocolo | Serviço / Função | Risco / Vetor de Ataque Comum |
| :--- | :--- | :--- | :--- |
| **88 (TCP/UDP)** | Kerberos v5 | Autenticação central por tickets (AS/TGS) | Kerberoasting, AS-REP Roasting, Golden Ticket |
| **389 (TCP/UDP)** | LDAP | Consultas ao catálogo e estrutura de objetos | Enumeração não autenticada (*null bind*), LDAP Injection |
| **636 (TCP)** | LDAPS | LDAP seguro criptografado com TLS | Forjamento de certificados corporativos (ADCS) |
| **445 (TCP)** | SMB | Compartilhamento de arquivos, RPC e IPC$ | Movimentação lateral (PsExec), EternalBlue, Pass-the-Hash |
| **53 (TCP/UDP)** | DNS | Resolução de nomes de controladores de domínio | Envenenamento de cache, DNS Zone Transfer |
| **135 (TCP)** | MSRPC / RPC | Comunicação e execução de procedimentos remotos | Enumeração de usuários via RPC (`rpcclient`, `enum4linux`) |

---

## Passo a Passo de Resolução

1. Execute o desafio para iniciar o módulo de avaliação:
   ```bash
   /challenge/ad-ldap-enumeration
   ```
2. Caso precise consultar a fundamentação teórica completa sobre Domain Controllers e LDAP:
   ```bash
   /challenge/ad-ldap-enumeration --help
   ```
3. O desafio apresentará quatro opções conceituais:
   * `[A]` Funcionar meramente como roteador TCP
   * `[B]` Autenticar usuários/serviços e centralizar a base mestra de segurança do domínio (NTDS.dit)
   * `[C]` Substituir switches de borda
   * `[D]` Converter IPv4 em IPv6 via NAT
4. Escolha a opção correta:
   ```text
   B
   ```
5. Você também pode disparar a resposta diretamente via CLI:
   ```bash
   /challenge/ad-ldap-enumeration B
   ```
6. O sistema homologará a resposta, apresentará o debriefing técnico e emitirá a flag.

---

## Dicas e Comandos Práticos

* **Enumeração LDAP no Linux via `ldapsearch`:**
  ```bash
  # Enumerar usuários que possuem SPN para Kerberoasting:
  ldapsearch -x -H ldap://dc01.corp.cyberguard.local \
    -b "DC=corp,DC=cyberguard,DC=local" \
    "(&(objectClass=user)(servicePrincipalName=*))" \
    sAMAccountName servicePrincipalName
  ```
* **Filtros LDAP de Alto Valor Ofensivo e Defensivo:**
  * Contas administrativas: `(&(objectCategory=person)(adminCount=1))`
  * Contas de serviço com delegação irrestrita: `(userAccountControl:1.2.840.113556.1.4.803:=524288)`
  * Contas sem exigência de pré-autenticação Kerberos: `(userAccountControl:1.2.840.113556.1.4.803:=4194304)`
* **Coleta Automatizada em Pentests com BloodHound / SharpHound:**
  O ingestor `SharpHound.exe` executa milhares de consultas LDAP paralelas em segundos para construir o grafo completo de relacionamentos corporativos.

---

## Aplicação Prática no Mundo Real
O comprometimento de Domain Controllers é o objetivo final de grupos criminosos de ransomware (como LockBit, BlackCat e Conti). Uma vez no DC, o adversário utiliza as Políticas de Grupo (GPO) para desativar antivírus e distribuir o ransomware simultaneamente em milhares de servidores e estações de trabalho da empresa em questão de minutos.

O modelo arquitetural moderno recomendado pela Microsoft (Enterprise Access Model / Tiering) isola os Domain Controllers no nível **Tier 0**, exigindo estações de trabalho dedicadas para administração (*PAWs - Privileged Access Workstations*) e autenticação exclusiva por chave de segurança física (FIDO2).
