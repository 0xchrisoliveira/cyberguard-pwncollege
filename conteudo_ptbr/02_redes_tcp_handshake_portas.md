# Guia 02 — Redes de Computadores: Handshake TCP, Portas e Tráfego
**Ciclo de Capacitação 2026.2 — Cyberguard Hub & Dojo pwn.college**  
**Classificação:** Educacional / Técnico  
**Público-Alvo:** Integrantes da Cyberguard (Ênfase nas lacunas de Redes, Handshake TCP e SSRF)

---

## 1. Introdução e Contexto do Diagnóstico

Na avaliação diagnóstica inicial da Cyberguard, **62,5% da equipe (5 de 8 membros)** falhou em identificar a sequência exata de estabelecimento de uma conexão TCP (o clássico *Three-Way Handshake*). 

O entendimento profundo do transporte TCP/IP não é apenas teoria acadêmica de redes: ele é a fundação para compreender técnicas como **Port Scanning furtivo (Nmap SYN scan)**, ataques de **Negação de Serviço (SYN Flood)**, evasão de firewalls e inspeção forense de pacotes com Wireshark.

---

## 2. O Modelo TCP/IP vs Modelo OSI

Para contextualizar a comunicação entre nós de rede, comparamos o modelo de referência conceitual OSI (7 camadas) com a implementação prática do mundo real, a pilha TCP/IP (4 camadas):

```
+---------------------------+---------------------------+---------------------------------+
| Camadas Modelo OSI        | Camadas TCP/IP            | Protocolos & Exemplos           |
+---------------------------+---------------------------+---------------------------------+
| 7. Aplicação              |                           | HTTP, HTTPS, SSH, DNS, FTP,     |
| 6. Apresentação           | 4. Aplicação              | SMTP, Kerberos, SMB             |
| 5. Sessão                 |                           |                                 |
+---------------------------+---------------------------+---------------------------------+
| 4. Transporte             | 3. Transporte             | TCP (confiável), UDP (rápido)   |
+---------------------------+---------------------------+---------------------------------+
| 3. Rede                   | 2. Internet               | IPv4, IPv6, ICMP, ARP           |
+---------------------------+---------------------------+---------------------------------+
| 2. Enlace de Dados        | 1. Acesso à Rede          | Ethernet, Wi-Fi (802.11),       |
| 1. Física                 |                           | Cabos de par trançado, Fibra    |
+---------------------------+---------------------------+---------------------------------+
```

* **UDP (User Datagram Protocol):** Orientado a datagramas, não confiável e sem confirmação de entrega. Excelente para streaming, jogos e consultas DNS rápidas.
* **TCP (Transmission Control Protocol):** Orientado à conexão, confiável, garante entrega ordenada, controle de fluxo e retransmissão de pacotes perdidos.

---

## 3. O Three-Way Handshake do TCP Detalhado

Antes de transmitir um único byte de dados via HTTP, SSH ou FTP, o cliente e o servidor devem estabelecer um canal sincronizado através de uma dança de três etapas conhecida como **Three-Way Handshake** (Aperto de Mão de Três Vias).

```
   CLIENTE (Ex: 192.168.1.10)                       SERVIDOR (Ex: 192.168.1.50:80)
        |                                                        |
        |  1. SYN (Flags: [SYN], Seq = X)                        |
        |------------------------------------------------------->|  Estado Servidor:
        |                                                        |  SYN_RECEIVED
        |                                                        |
        |  2. SYN-ACK (Flags: [SYN, ACK], Seq = Y, Ack = X + 1)  |
        |<-------------------------------------------------------|
        |                                                        |
Estado Cliente:                                                  |
ESTABLISHED                                                      |
        |                                                        |
        |  3. ACK (Flags: [ACK], Seq = X + 1, Ack = Y + 1)       |
        |------------------------------------------------------->|  Estado Servidor:
        |                                                        |  ESTABLISHED
        |                                                        |
        |========== CONEXÃO ESTABELECIDA / TRANSFERÊNCIA ========|
```

### 3.1 As Três Etapas Passo a Passo:

1. **Passo 1 — SYN (Synchronize):**
   * O cliente envia um pacote TCP com a flag **SYN** ativada.
   * Ele escolhe um número de sequência inicial aleatório (**ISN — Initial Sequence Number**), chamado aqui de `X` (`Seq = X`).
   * O cliente entra no estado `SYN_SENT`.
2. **Passo 2 — SYN-ACK (Synchronize-Acknowledgment):**
   * O servidor responde reconhecendo o pedido do cliente.
   * Ele ativa as flags **SYN** e **ACK** simultaneamente.
   * Envia seu próprio número de sequência inicial aleatório `Y` (`Seq = Y`).
   * Reconhece o número do cliente somando 1: `Ack = X + 1` (significa: "recebi X, aguardo o byte X+1").
   * O servidor entra no estado `SYN_RECEIVED`.
3. **Passo 3 — ACK (Acknowledgment):**
   * O cliente confirma o recebimento do SYN do servidor.
   * Envia a flag **ACK** com `Seq = X + 1` e `Ack = Y + 1`.
   * Ambos os lados entram no estado `ESTABLISHED`. Agora a troca de dados da camada de aplicação pode começar.

### 3.2 O Papel das Principais Flags TCP:

* **SYN (Synchronize):** Inicia e sincroniza uma nova conexão.
* **ACK (Acknowledgment):** Confirma o recebimento bem-sucedido de dados anteriores.
* **FIN (Finish):** Solicita o encerramento gracioso e ordenado de uma conexão.
* **RST (Reset):** Aborta a conexão imediatamente (usado quando uma porta está fechada ou por firewalls).
* **PSH (Push):** Solicita que a camada TCP envie os dados acumulados imediatamente para a aplicação, sem aguardar o preenchimento de todo o buffer.
* **URG (Urgent):** Sinaliza dados prioritários a serem processados fora de ordem.

---

## 4. Tabela de Portas e Protocolos Clássicos

Em operações de Red Team e monitoramento Blue Team, certas portas devem ser reconhecidas instantaneamente:

| Porta | Protocolo | Camada de Transporte | Descrição / Uso Típico |
| :---: | :---: | :---: | :--- |
| **21** | FTP | TCP | Transferência de arquivos (credenciais em texto plano) |
| **22** | SSH | TCP | Acesso remoto seguro e criptografado |
| **23** | Telnet | TCP | Acesso remoto legro (inseguro, sem criptografia) |
| **25** | SMTP | TCP | Envio de e-mails entre servidores |
| **53** | DNS | UDP & TCP | Resolução de nomes de domínio (TCP para transferência de zona) |
| **80** | HTTP | TCP | Tráfego web não criptografado |
| **88** | Kerberos | UDP & TCP | Autenticação em ambientes Active Directory |
| **110** | POP3 | TCP | Recuperação de e-mails em clientes locais |
| **139 / 445** | SMB | TCP | Compartilhamento de arquivos e named pipes em redes Windows |
| **143** | IMAP | TCP | Sincronização de caixas postais |
| **389 / 636** | LDAP / LDAPS | TCP | Consulta a diretórios de usuários e objetos corporativos |
| **443** | HTTPS | TCP | Tráfego web criptografado via TLS/SSL |
| **1433** | MS SQL | TCP | Banco de dados Microsoft SQL Server |
| **3306** | MySQL / MariaDB | TCP | Banco de dados relacional amplamente utilizado |
| **3389** | RDP | TCP | Remote Desktop Protocol (Área de Trabalho Remota Windows) |
| **5432** | PostgreSQL | TCP | Banco de dados relacional avançado |
| **6379** | Redis | TCP | Banco de dados NoSQL chave-valor em memória |
| **8080 / 8443** | HTTP-Alt | TCP | Portas alternativas para servidores web e proxies |

---

## 5. Ferramentas Práticas Essenciais

### 5.1 O Canivete Suíço: `netcat` (`nc`)

O Netcat lê e escreve dados através de conexões de rede utilizando os protocolos TCP ou UDP.

```bash
# 1. Escuta passiva em uma porta (Listener para Reverse Shell):
nc -lvnp 4444
# -l: Modo de escuta (listen)
# -v: Modo verboso
# -n: Não resolver nomes via DNS (mais rápido)
# -p: Número da porta local

# 2. Conexão a um serviço para Banner Grabbing (identificação de versão):
nc -vn 192.168.1.50 22
# Retorna: SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.1

# 3. Transferência simples de arquivo entre máquinas:
# No receptor:
nc -lvnp 9001 > dump_recebido.bin
# No remetente:
nc -vn 192.168.1.100 9001 < /tmp/dados_sensiveis.bin
```

### 5.2 Captura de Pacotes na Linha de Comando: `tcpdump`

```bash
# Capturar 50 pacotes na interface eth0 filtrando apenas tráfego HTTP na porta 80:
tcpdump -i eth0 -nn -c 50 'tcp port 80'

# Capturar pacotes com visualização em ASCII e Hexadecimal (ótimo para ler payloads HTTP):
tcpdump -i eth0 -A -s 0 'tcp port 80'

# Salvar pacotes diretamente para arquivo .pcap para análise no Wireshark:
tcpdump -i eth0 -w captura_incidente.pcap 'tcp port 445 or tcp port 88'
```

### 5.3 Análise Visual com Wireshark

O Wireshark decompõe todos os campos dos cabeçalhos Ethernet, IP e TCP. Filtros de exibição (*Display Filters*) indispensáveis:

```wireshark
# Localizar o início de conexões (pacotes SYN puros):
tcp.flags.syn == 1 && tcp.flags.ack == 0

# Localizar conexões rejeitadas ou abortadas (flag RST ativada):
tcp.flags.reset == 1

# Filtrar tráfego para uma porta específica:
tcp.port == 80 || tcp.port == 443

# Seguir a conversa completa em texto legível:
Botão direito no pacote -> "Follow" -> "TCP Stream"
```

### 5.4 Varredura e Auditoria com `nmap`

O Nmap utiliza a manipulação de flags TCP para inferir o estado das portas remotas:

```bash
# 1. SYN Scan (Stealth / Half-Open Scan - Opção padrão com privilégios de root):
nmap -sS -p- -T4 10.10.10.50
# Mecânica: O Nmap envia um SYN. Se o alvo responde SYN-ACK, a porta está aberta.
# Em vez de enviar o ACK final (completando a conexão), o Nmap envia um RST!
# Vantagem: Mais rápido e frequentemente não registrado por logs de aplicações antigas.

# 2. Enumeração completa com detecção de versão (-sV) e scripts NSE padrão (-sC):
nmap -sV -sC -p 21,22,80,445 10.10.10.50
```

---

## 6. Introdução ao SSRF no Contexto de Rede

O **SSRF (Server-Side Request Forgery)** ocorre quando um invasor consegue forçar uma aplicação web hospedada no servidor a realizar requisições de rede para destinos escolhidos por ele.

```
+------------+       Requisição HTTP Externa        +--------------------+
|  Invasor   | -----------------------------------> | Servidor Web (DMZ) |
+------------+   url=http://127.0.0.1:6379/flushall +--------------------+
                                                              |
                                                              | Conexão Interna
                                                              | (Sem Firewall)
                                                              v
                                                    +--------------------+
                                                    |  Redis Interno     |
                                                    |  127.0.0.1:6379    |
                                                    +--------------------+
```

### Por que o SSRF é tão Devastador em Redes Corporativas?

1. **Quebra do Perímetro e Acesso ao Localhost (`127.0.0.1`):**  
   Muitos microsserviços internos, painéis administrativos (`localhost:8080/admin`) ou bancos em memória como **Redis** (`port 6379`) são configurados sem autenticação porque os desenvolvedores assumem que apenas processos locais terão acesso. Se a aplicação web possui SSRF, o atacante usa a própria aplicação como procurador (*proxy*) para interagir diretamente com esses serviços restritos.
2. **O Endereço Mágico da Nuvem: `169.254.169.254` (Link-Local):**  
   Em provedores de nuvem como **AWS, Google Cloud e Microsoft Azure**, o IP `169.254.169.254` é reservado para o serviço de metadados da instância (IMDS). Se o invasor explora um SSRF em uma aplicação rodando na AWS:
   ```bash
   GET /proxy?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/WebServerRole
   ```
   A aplicação buscará as credenciais temporárias da role IAM (AccessKeyId, SecretAccessKey e Token) e as entregará de bandeja para o atacante, permitindo o comprometimento de toda a conta de nuvem!
