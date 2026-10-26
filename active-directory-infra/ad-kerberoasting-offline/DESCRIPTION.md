# Identificação de SPNs e Ataque Kerberoasting (AD Kerberoasting Offline)

## Briefing da Missão
* **Codinome:** Operação Golden Roast
* **Classificação:** Confidencial / Treinamento de Red Team & Active Directory Attacks
* **Alvo:** Contas de Serviço com Service Principal Names (SPN) Cadastrados
* **Objetivo:** Compreender a mecânica do ataque de Kerberoasting (MITRE ATT&CK T1558.003), a geração de tickets TGS no Key Distribution Center (KDC) e os métodos de quebra offline de credenciais sem disparar bloqueios no Domain Controller.

No teste diagnóstico de conhecimentos prévios do ciclo 2026.2 da Cyberguard, a **questão 07** evidenciou uma defasagem crítica: **75% da equipe (6 de 8 membros)** errou a definição de Kerberoasting, com respondentes confundindo o ataque com uma negação de serviço (DoS) contra servidores DNS. Este laboratório prático elucida o vetor de ataque mais prevalente em ambientes corporativos Windows.

---

## Teoria Aprofundada

### 1. A Falha Estrutural do Protocolo Kerberos v5
O protocolo Kerberos v5 permite que clientes autentiquem em serviços (como bancos MSSQL, servidores web IIS ou instâncias de mensageria) sem trafegar senhas pela rede.

Contudo, para garantir compatibilidade e usabilidade:
> **Qualquer conta de usuário autenticada no domínio — mesmo o usuário comum mais restrito da rede — possui o direito de solicitar um Ticket de Serviço (TGS) para QUALQUER conta de serviço que tenha um SPN registrado!**

E o mais crítico: o Domain Controller (KDC) **não valida** se o solicitante possui permissão real para usar aquele serviço. O KDC simplesmente emite o bilhete TGS criptografado com o hash NTLM da conta proprietária do serviço.

### 2. Diagrama do Fluxo de Kerberoasting

```
[ ATACANTE (Acesso Comum) ]            [ DOMAIN CONTROLLER (KDC) ]         [ SERVIDOR MSSQL ]
         │                                          │                                 │
         │ 1. Consulta LDAP: "Quais contas          │                                 │
         │    possuem SPN registrado?"              │                                 │
         ├─────────────────────────────────────────>│                                 │
         │                                          │                                 │
         │ 2. Envia TGS-REQ pedindo ticket para     │                                 │
         │    MSSQLSvc/sql01.corp.local:1433        │                                 │
         ├─────────────────────────────────────────>│                                 │
         │                                          │                                 │
         │ 3. KDC emite TGS-REP: Ticket cifrado     │                                 │
         │    com o HASH DA CONTA DO SPN!           │                                 │
         │<─────────────────────────────────────────┤                                 │
         │                                                                            │
         │ 4. ATACANTE NÃO CONECTA NO SERVIDOR ALVO!                                  │
         │    O ticket é extraído da memória local e submetido                        │
         │    a FORÇA BRUTA OFFLINE no Hashcat / John.                                │
         │                                                                            │
         ▼                                                                            │
[ QUEBRA 100% OFFLINE ] ──> Senha Fraca Revelada!                                     │
         │                                                                            │
         │ 5. Login direto com as credenciais da conta de serviço!                    │
         └───────────────────────────────────────────────────────────────────────────>│
```

### 3. Por que o Ataque é Invisível para Controles Tradicionais de Bloqueio?
Quando um invasor tenta chutar senhas contra um formulário web ou conexão SMB, cada tentativa errada incrementa o contador `badPwdCount` no Domain Controller, bloqueando a conta após poucas tentativas (ex: 5 erros).

No **Kerberoasting**:
* O invasor solicita o bilhete ao DC **apenas uma vez** (uma operação legítima que não gera log de erro).
* Todas as milhões ou bilhões de tentativas de senha do dicionário ocorrem **offline**, nos núcleos de GPU do computador do atacante.
* O Domain Controller nunca fica sabendo que um ataque de força bruta está em andamento.

### 4. Tabela de Comparação: Kerberoasting vs AS-REP Roasting

| Critério | Kerberoasting (T1558.003) | AS-REP Roasting (T1558.004) |
| :--- | :--- | :--- |
| **Alvo Principal** | Contas de serviço com atributo `servicePrincipalName` | Contas com flag `DONT_REQ_PREAUTH` (sem pré-autenticação) |
| **Ticket Criptografado** | Ticket TGS (emitido pelo Ticket Granting Service) | Resposta AS-REP (emitida pelo Authentication Service) |
| **Autenticação Prévia** | Exige conta válida no domínio | **Não exige conta** (pode ser disparado externamente) |
| **Chave de Criptografia**| Hash NTLM da conta do SPN | Hash NTLM da própria conta sem pré-autenticação |
| **Modo no Hashcat** | `-m 13100` (Kerberos 5 TGS-REP etype 23) | `-m 18200` (Kerberos 5 AS-REP etype 23) |

---

## Passo a Passo de Resolução

1. Execute o binário do desafio de auditoria Kerberos:
   ```bash
   /challenge/ad-kerberoasting-offline
   ```
2. Caso deseje consultar a aula técnica detalhada sobre Kerberoasting:
   ```bash
   /challenge/ad-kerberoasting-offline --help
   ```
3. O desafio exibirá as opções conceituais sobre o mecanismo técnico do ataque:
   * Opção 1: Esgotamento de conexões SYN no servidor DNS
   * Opção 2: Solicitar tickets de serviço (TGS) de contas com SPN ao KDC e quebrar o hash offline
   * Opção 3: Injeção SQL em estações de trabalho sem rede
   * Opção 4: Clonagem de endereço MAC de switch core
4. Selecione a opção `2`:
   ```text
   2
   ```
5. Você também pode disparar a execução diretamente com o parâmetro CLI:
   ```bash
   /challenge/ad-kerberoasting-offline 2
   ```
6. O sistema confirmará a assimilação do ataque, emitirá o debriefing de defesa e liberará a flag.

---

## Dicas e Comandos Práticos

* **Requisitando Tickets TGS com Impacket (Linux):**
  ```bash
  # Solicita e exporta os tickets no formato do Hashcat ($krb5tgs$23$...):
  GetUserSPNs.py corp.cyberguard.local/aluno:Senha123 \
    -dc-ip 10.10.10.1 \
    -request \
    -outputfile kerberoast_hashes.txt
  ```
* **Quebrando Tickets no Hashcat:**
  ```bash
  # Modo 13100 para TGS-REP (RC4 / etype 23):
  hashcat -m 13100 -a 0 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt
  ```
* **No Windows (via PowerShell e PowerView / Rubeus):**
  ```powershell
  # Extrair e requisitar tickets diretamente na memória:
  .\Rubeus.exe kerberoast /outfile:hashes.txt
  ```

---

## Aplicação Prática no Mundo Real
O Kerberoasting é uma das técnicas mais documentadas em relatórios de pentest e incidentes de Red Team porque contas de serviço geralmente:
1. Possuem privilégios administrativos elevados (ex: membros de `Administrators` ou `Server Operators`).
2. Possuem senhas antigas configuradas com a política *"Password Never Expires"* para evitar interrupção de sistemas críticos de banco de dados.

**A Defesa Canônica no Active Directory:**
* Migrar todas as contas de serviço para **gMSA (Group Managed Service Accounts)**: o Windows passa a gerenciar senhas complexas de 128 caracteres aleatórios, rotacionadas a cada 30 dias automaticamente.
* Monitorar no SIEM o evento **Event ID 4769** com algoritmo de criptografia `0x17` (RC4), indicando potencial tentativa de downgrade de cifragem para facilitar o cracking.
