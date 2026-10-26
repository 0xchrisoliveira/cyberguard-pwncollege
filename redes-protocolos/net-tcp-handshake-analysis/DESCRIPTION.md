# Dissecando o Handshake TCP no Wireshark e Linha de Comando

## 1. Briefing da Missão
Operador, durante a auditoria forense de conexões maliciosas interceptadas pelo SOC da Cyberguard, é mandatório distinguir com precisão fluxos de conexões TCP estabelecidas com sucesso de varreduras incompletas (*half-open scans*) e tentativas de negação de serviço (*SYN floods*).

Sua missão tática é:
1. Compreender a anatomia do aperto de mão de três vias do TCP (*Three-Way Handshake*).
2. Compreender os estados dos sockets do kernel (`LISTEN`, `SYN_SENT`, `SYN_RECV`, `ESTABLISHED`).
3. Interagir com o validador `/challenge/net-tcp-handshake-analysis`, comprovar o domínio do ciclo de vida das flags TCP e capturar a flag.

---

## 2. Teoria Aprofundada

### 2.1 O Modelo do Three-Way Handshake (RFC 793)
O protocolo TCP (Transmission Control Protocol) é orientado a conexão. Diferente do UDP, nenhum byte de aplicação é transmitido antes que ambos os lados sincronizem seus números de sequência iniciais (*ISN — Initial Sequence Numbers*).

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

### 2.2 Tabela de Flags de Controle TCP

| Flag TCP | Bit Hexadecimal | Função Primária | Cenário de Segurança |
| :---: | :---: | :--- | :--- |
| **SYN** | `0x02` | Sincroniza números de sequência para iniciar canal | Início de conexão / SYN Flood DoS |
| **ACK** | `0x10` | Reconhece o recebimento ordenado de bytes anteriores | Confirmação de pacote / ACK scan |
| **FIN** | `0x01` | Solicita término elegante da conexão | Fechamento de sessão de aplicação |
| **RST** | `0x04` | Aborta a conexão imediatamente | Porta fechada / Encerramento do SYN scan |
| **PSH** | `0x08` | Força envio imediato dos dados do buffer para a aplicação | Envio de comandos interativos (SSH/Telnet) |
| **URG** | `0x20` | Marca o ponteiro de dados urgentes | Priorização de interrupções |

### 2.3 Raio-X do Diagnóstico da Cyberguard (Questão Q02)
Na avaliação diagnóstica de entrada do Ciclo 2026.2, **62,5% da equipe falhou na identificação do Three-Way Handshake**. Integrantes confundiram a posição do `ACK` ou assumiram que o cliente enviava `SYN-ACK`. 

O mnemônico infalível é:
* **Cliente:** Começa com **SYN** (solicita sincronismo).
* **Servidor:** Responde com **SYN-ACK** (sincroniza o dele e reconhece o seu).
* **Cliente:** Conclui com **ACK** (reconhece o do servidor).

---

## 3. Passo a Passo de Resolução

1. **Invoque o modo de ajuda para revisar a teoria:**
   ```bash
   /challenge/net-tcp-handshake-analysis -h
   ```
   *Leia o diagrama e a tabela de flags.*

2. **Execute o binário do desafio:**
   ```bash
   /challenge/net-tcp-handshake-analysis
   ```

3. **Selecione a alternativa correspondente:**
   Quando solicitado:
   ```text
   [?] Digite o número da alternativa correta (1, 2, 3 ou 4):
   ```
   Digite `2` (correspondente a `SYN -> SYN-ACK -> ACK`) e pressione `Enter`.

4. **Alternativa via argumento direto:**
   ```bash
   /challenge/net-tcp-handshake-analysis 2
   ```

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição |
| :--- | :--- |
| `ss -ant` | Exibe todas as conexões TCP numéricas e seus estados (`ESTAB`, `SYN-SENT`, `LISTEN`). |
| `netstat -nat` | Comando legado equivalente ao `ss` para listagem de conexões ativas. |
| `tcpdump -nn "tcp[tcpflags] == tcp-syn"` | Captura exclusivamente pacotes de início de conexão (SYN puro). |
| `tcpdump -nn "tcp[tcpflags] & (tcp-rst) != 0"` | Filtra conexões que foram resetadas/abortadas bruscamente. |

---

## 5. Aplicação Prática no Mundo Real

* **Evasão e Detecção de Port Scans (Nmap Stealth):** O famoso scan `-sS` do Nmap envia o pacote `SYN`, recebe o `SYN-ACK` e imediatamente envia um `RST` em vez do `ACK` final. A porta é categorizada como aberta, mas o servidor nunca completa o handshake, impedindo que daemons de aplicação registrem a conexão em seus logs de auditoria.
* **Mitigação de Ataques SYN Flood:** Servidores protegidos utilizam o recurso de *SYN Cookies* no kernel Linux (`sysctl -w net.ipv4.tcp_syncookies=1`). O servidor codifica o estado da conexão diretamente no número de sequência `ISN` sem alocar memória na tabela até que o `ACK` legítimo retorne.
