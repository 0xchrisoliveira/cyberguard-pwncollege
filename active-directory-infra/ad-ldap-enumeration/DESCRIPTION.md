# Enumeração de Objetos via LDAP

### 📖 Introdução Teórica
Em mais de 90% das redes corporativas de grande e médio porte no mundo, a gestão de identidades, autenticações e permissões é unificada pelo **Active Directory Domain Services (AD DS)** da Microsoft.

**O Papel Central do Domain Controller (DC):**
O **Domain Controller (DC)** é o servidor mais crítico de toda a infraestrutura corporativa. Ele não é um mero comutador ou roteador de rede. O DC é a **âncora de segurança e autoridade primária do domínio**:
1. **Autenticação Centralizada:** Valida a identidade de cada colaborador, computador e serviço da organização através de protocolos como Kerberos v5 e NTLM.
2. **A Base de Dados Mestra (`NTDS.dit`):** O DC armazena e gerencia o arquivo `C:\Windows\NTDS\NTDS.dit`. Essa base de dados contém todos os objetos do domínio corporativo: contas de usuários, grupos de segurança, computadores, relações de confiança e, mais criticamente, **os hashes de senha (NTLM) e segredos Kerberos de todas as contas corporativas**, incluindo o administrador do domínio (`Domain Admins`) e a conta `krbtgt` (responsável pela criptografia de tickets TGT).
3. **Distribuição de Políticas (GPOs):** Aplica políticas de segurança uniformes em milhares de estações de trabalho e servidores com um único clique.
4. **Serviços Essenciais de Suporte:** Opera o Key Distribution Center (KDC Kerberos na porta 88), servidores de nomes DNS (porta 53) e catálogo de diretório LDAP (portas 389 e 636).

**O Protocolo LDAP (Lightweight Directory Access Protocol):**
O LDAP é o protocolo padrão da indústria utilizado para consultar e manipular objetos dentro da árvore do Active Directory.
* **Por que a enumeração LDAP é o primeiro passo de qualquer atacante ou auditor?**
  Ao obter acesso a qualquer máquina conectada ao domínio (ou em cenários com consultas anônimas / *null bind* permitidas), o atacante utiliza o LDAP para mapear toda a topologia corporativa:
  - Identificar contas com privilégios administrativos (`adminCount=1`).
  - Localizar contas de serviço que possuem Service Principal Names (SPNs) configurados (alvos diretos de *Kerberoasting*).
  - Encontrar contas sem exigência de pré-autenticação Kerberos (alvos de *AS-REP Roasting*).
  - Descobrir estações com sistemas operacionais obsoletos e grupos delegados de HelpDesk.

Na segurança defensiva (Blue Team), a proteção do Domain Controller é classificada como **Tier 0** (nível de isolamento máximo). Se um adversário obtiver credenciais de um DC ou replicar a base `NTDS.dit` via técnica de `DCSync` (`secretsdump.py`), toda a empresa estará irremediavelmente comprometida.

### 💻 Exemplos Práticos e Sintaxe
Consultas LDAP essenciais no terminal Linux utilizando `ldapsearch`:

1. **Conexão básica e enumeração da raiz do domínio (`-x` para bind simples):**
   ```bash
   ldapsearch -x -H ldap://dc01.corp.cyberguard.local      -b "DC=corp,DC=cyberguard,DC=local"      "(objectClass=organizationalPerson)" sAMAccountName mail
   ```

2. **Filtrar contas de serviço que possuem SPN (Alvos de Kerberoasting):**
   ```bash
   ldapsearch -x -H ldap://dc01.corp.cyberguard.local      -b "DC=corp,DC=cyberguard,DC=local"      "(&(objectClass=user)(servicePrincipalName=*))"      sAMAccountName servicePrincipalName
   ```
   *Saída esperada:*
   ```text
   # srv_sql, Users, corp.cyberguard.local
   dn: CN=srv_sql,CN=Users,DC=corp,DC=cyberguard,DC=local
   sAMAccountName: srv_sql
   servicePrincipalName: MSSQLSvc/sql01.corp.cyberguard.local:1433
   ```

3. **Filtrar usuários com flag de privilégio administrativo (`adminCount=1`):**
   ```bash
   ldapsearch -x -H ldap://dc01.corp.cyberguard.local      -b "DC=corp,DC=cyberguard,DC=local"      "(&(objectCategory=person)(adminCount=1))"      sAMAccountName memberOf
   ```

4. **Filtrar contas vulneráveis a AS-REP Roasting (bit DONT_REQ_PREAUTH):**
   ```bash
   ldapsearch -x -H ldap://dc01.corp.cyberguard.local      -b "DC=corp,DC=cyberguard,DC=local"      "(userAccountControl:1.2.840.113556.1.4.803:=4194304)"      sAMAccountName
   ```

### 🎯 Seu Objetivo

Enumerar informações estruturais de um domínio Active Directory utilizando consultas LDAP. Seu objetivo é consultar a árvore de diretórios para descobrir contas de usuários com privilégios elevados, mapear o grupo de Domain Admins e identificar propriedades e descrições de contas que revelem informações confidenciais para responder à auditoria do desafio.
