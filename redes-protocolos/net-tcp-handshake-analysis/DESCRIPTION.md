# Dissecando o Handshake TCP no Wireshark/Tshark (Anatomia do Three-Way Handshake)

### 📖 Introdução Teórica

Diferente do protocolo UDP (User Datagram Protocol), que opera sob o princípio do "melhor esforço" sem confirmação de entrega, o **TCP (Transmission Control Protocol — RFC 793)** é um protocolo estritamente orientado a conexão. Isso significa que nenhum byte de dados da Camada de Aplicação pode ser transmitido antes que o cliente e o servidor sincronizem seus parâmetros de canal através de um procedimento de três etapas conhecido como **Three-Way Handshake**.

```
   CLIENTE (Ex: 192.168.1.10)                       SERVIDOR (Ex: 192.168.1.50:80)
        │                                                        │
        │  1. SYN (Flags: [SYN], Seq = X)                        │
        │───────────────────────────────────────────────────────>│  Estado Servidor:
        │                                                        │  SYN_RECEIVED
        │                                                        │
        │  2. SYN-ACK (Flags: [SYN, ACK], Seq = Y, Ack = X + 1)  │
        │<───────────────────────────────────────────────────────│
        │                                                        │
Estado Cliente:                                                  │
ESTABLISHED                                                      │
        │                                                        │
        │  3. ACK (Flags: [ACK], Seq = X + 1, Ack = Y + 1)       │
        │───────────────────────────────────────────────────────>│  Estado Servidor:
        │                                                        │  ESTABLISHED
        │                                                        │
        │══════════ CONEXÃO ESTABELECIDA / TRANSFERÊNCIA ════════│
```

#### As Três Etapas Rigorosas do Handshake
1. **SYN (Synchronize):** O cliente envia um pacote com a flag `SYN` ativada e um número de sequência inicial pseudoaleatório (*ISN = X*). Indica formalmente a intenção de abrir um canal de comunicação.
2. **SYN-ACK (Synchronize-Acknowledgment):** O servidor responde com as flags `SYN` e `ACK` ativadas. Ele confirma o recebimento do pacote do cliente incrementando seu número de sequência (*Ack = X + 1*) e introduz seu próprio número de sequência inicial (*Seq = Y*).
3. **ACK (Acknowledgment):** O cliente responde com a flag `ACK` ativada, confirmando o número de sequência do servidor (*Ack = Y + 1*). A partir deste instante, ambos os nós entram no estado `ESTABLISHED` e a troca de dados em texto claro ou criptografado tem início.

#### Tabela de Flags de Controle do Cabeçalho TCP
| Flag TCP | Bit Hexadecimal | Propósito Operacional | Cenário em Cibersegurança |
| :---: | :---: | :--- | :--- |
| **SYN** | `0x02` | Sincroniza números de sequência para abrir canal | Início de conexões legítimas e ataques SYN Flood |
| **ACK** | `0x10` | Reconhece o recebimento ordenado de pacotes | Confirmação contínua de entrega de segmentos |
| **FIN** | `0x01` | Solicita o encerramento gracioso da conexão | Finalização ordenada de sessões TCP |
| **RST** | `0x04` | Aborta a conexão imediatamente (*Reset*) | Portas fechadas ou encerramento de SYN scan |
| **PSH** | `0x08` | Força envio imediato dos dados do buffer para a aplicação | Comandos interativos em terminais SSH e shells |
| **URG** | `0x20` | Informa a presença de dados de processamento urgente | Priorização de interrupções de controle |

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team):** 
  - **SYN Scans (Stealth Scan):** Ao realizar um scan com `nmap -sS`, o atacante envia o pacote `SYN`, recebe o `SYN-ACK` e, em vez de responder com o `ACK` final, envia um `RST`. Isso determina que a porta está aberta sem concluir a conexão, evitando que serviços legados registrem a tentativa em seus arquivos de log.
  - **SYN Flood (DDoS):** Atacantes enviam milhões de pacotes `SYN` com IPs de origem forjados (*spoofing*), saturando a tabela de estados (`SYN_RECV`) do alvo e causando negação de serviço.
* **Segurança Defensiva (Blue Team / SOC):** Analistas utilizam filtros no Wireshark e Zeek para detectar discrepâncias em flags (como pacotes anômalos com flags `SYN+FIN`), monitorar o tempo de resposta do handshake e configurar contramedidas no kernel como **SYN Cookies** (`net.ipv4.tcp_syncookies = 1`).

---

### 💻 Exemplos Práticos e Sintaxe

A inspeção e o diagnóstico do handshake TCP no Linux dependem dos utilitários de inspeção de sockets e sniffers:

#### Comandos de Inspeção e Filtros
* `ss -ant`: Exibe todas as conexões TCP numéricas (`-n`) ativas e em escuta (`-a`), indicando seu estado atual (`ESTAB`, `SYN-SENT`, `LISTEN`).
* `tcpdump -nn "tcp[tcpflags] == tcp-syn"`: Filtra exclusivamente os pacotes de solicitação inicial de conexão (onde apenas o bit SYN está ativo).
* `tcpdump -nn "tcp[tcpflags] & (tcp-syn|tcp-ack) == (tcp-syn|tcp-ack)"`: Filtra especificamente as respostas SYN-ACK dos servidores.
* Filtros de Display no Wireshark:
  * `tcp.flags.syn == 1 && tcp.flags.ack == 0`: Identifica pacotes SYN isolados (início).
  * `tcp.flags.syn == 1 && tcp.flags.ack == 1`: Identifica pacotes SYN-ACK (resposta do servidor).
  * `tcp.flags.reset == 1`: Identifica conexões rejeitadas ou abortadas com RST.

#### Snippets de Terminal e Saídas Observadas

1. **Visualizando Estados de Conexão no Linux com `ss`:**
   ```bash
   $ ss -ant
   State      Recv-Q Send-Q  Local Address:Port   Peer Address:Port
   LISTEN     0      128         127.0.0.1:80           0.0.0.0:*
   ESTAB      0      0       192.168.1.10:45678    192.168.1.50:80
   SYN-SENT   0      1       192.168.1.10:45680    192.168.1.50:443
   ```

2. **Interceptando o Handshake Completo via Tcpdump:**
   ```text
   12:00:00.100 IP 10.0.0.2.4000 > 10.0.0.1.80: Flags [S], seq 1000
   12:00:00.105 IP 10.0.0.1.80 > 10.0.0.2.4000: Flags [S.], seq 2000, ack 1001
   12:00:00.106 IP 10.0.0.2.4000 > 10.0.0.1.80: Flags [.], ack 2001
   ```
   *Note a notação compacta do tcpdump: `[S]` = SYN, `[S.]` = SYN-ACK, `[.]` = ACK puro.*

---

### 🎯 Seu Objetivo

Analisar detalhadamente o mecanismo de sincronização de conexões TCP (Three-Way Handshake: SYN -> SYN-ACK -> ACK). Seu objetivo é executar `/challenge/net-tcp-handshake-analysis` e responder corretamente às perguntas sobre as flags ativas, números de sequência (`SEQ`) e números de confirmação (`ACK`) durante a negociação da conexão para validar seus conhecimentos teóricos e resgatar a flag.
