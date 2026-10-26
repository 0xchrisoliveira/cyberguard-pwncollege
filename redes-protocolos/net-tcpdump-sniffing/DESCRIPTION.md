# Sniffing de Pacotes com Tcpdump (Análise de Tráfego & Filtros BPF)

## 1. Briefing da Missão
Operador, a equipe de segurança da Cyberguard detectou transmissões UDP periódicas trafegando pela interface de loopback (`lo`) de uma máquina de produção. Suspeita-se de um canal de telemetria desprotegido vazando tokens confidenciais em texto plano.

Sua missão tática é:
1. Posicionar um sniffer de pacotes de rede na interface local `lo`.
2. Aplicar filtros BPF para isolar o tráfego que transita na porta `44444`.
3. Inspecionar o payload dos pacotes em formato legível (ASCII).
4. Submeter o token capturado ao verificador `/challenge/net-tcpdump-sniffing` e resgatar a flag.

---

## 2. Teoria Aprofundada

### 2.1 Como Funciona a Captura de Pacotes no Kernel Linux
Normalmente, a pilha de rede do Linux descarta qualquer pacote que não seja endereçado a portas com listeners ativos. No entanto, quando um utilitário como o `tcpdump` é acionado:
1. Ele cria um socket de camada de enlace (`socket(AF_PACKET, SOCK_RAW, ...)`).
2. O subsistema do kernel cria uma cópia de todos os quadros e datagramas recebidos ou emitidos pela interface de rede especificada.
3. Esses dados são passados para o motor do **Berkeley Packet Filter (BPF)**, que roda diretamente no kernel space para descartar pacotes que não batem com a expressão fornecida pelo usuário, economizando ciclos de CPU e memória.

```
       [ Aplicação Emissora: net-tcpdump-sniffing ]
                         │ (sendto UDP port 44444)
                         ▼
             [ Interface de Loopback 'lo' ]
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
      [ Pilha TCP/IP IPV4 ]     [ AF_PACKET Raw Socket ]
                                     │
                                     ▼
                            [ Filtro BPF Kernel ] (udp port 44444)
                                     │
                                     ▼
                              [ tcpdump -i lo -A ]
                                     │
                                     ▼
                          [ Payload em Texto Claro ]
```

### 2.2 Tabela de Parâmetros Críticos do Tcpdump

| Flag | Nome / Função | Por que é importante? |
| :---: | :--- | :--- |
| **`-i lo`** | Interface de escuta | Especifica o loopback local (`127.0.0.1`). Sem ela, o tcpdump escuta `eth0`. |
| **`-A`** | ASCII Output | Renderiza os dados do pacote como caracteres legíveis no terminal. |
| **`-X`** | Hex + ASCII | Exibe offset, bytes em hexadecimal e caracteres ASCII lado a lado. |
| **`-nn`** | No DNS / No Port Names | Não resolve IPs para domínios nem converte portas em nomes (ex: 80 -> http). Acelera absurdamente. |
| **`-s 0`** | Snapshot Length | Captura o pacote inteiro sem truncar (em versões modernas já é padrão). |
| **`-w cap.pcap`** | Write to file | Grava os pacotes em formato PCAP para análise posterior no Wireshark. |

### 2.3 Relação com o Diagnóstico da Cyberguard
No relatório diagnóstico da Cyberguard, observou-se que analistas com viés estritamente teórico de SIEM tinham dificuldade em operar na ponta da linha em shells de Linux durante incidentes ao vivo. Dominar o `tcpdump` é mandatório quando não há interface gráfica (GUI) disponível em servidores remotos.

---

## 3. Passo a Passo de Resolução

1. **Abra um terminal dedicado para a captura:**
   Inicie o `tcpdump` escutando a interface local com filtro na porta UDP 44444:
   ```bash
   sudo tcpdump -i lo -A -nn udp port 44444
   ```
   *Se estiver no container pwn.college com permissões padrão:*
   ```bash
   tcpdump -i lo -A -nn udp port 44444
   ```

2. **Em outro terminal, inicie o emissor de tráfego:**
   ```bash
   /challenge/net-tcpdump-sniffing
   ```
   *O binário iniciará o envio cíclico dos pacotes a cada 800 milissegundos e aguardará sua resposta no prompt.*

3. **Inspecione o terminal do tcpdump:**
   Você verá linhas semelhantes a:
   ```text
   12:34:56.789012 IP 127.0.0.1.54321 > 127.0.0.1.44444: UDP, length 42
   E..J..@.@.V...........4...~.CYBERGUARD_SNIFF_SECRET=tcpdump_mestre_2026
   ```

4. **Copie o token e insira no desafio:**
   O token é a parte após o sinal de igual:
   ```text
   tcpdump_mestre_2026
   ```
   Cole no prompt do `/challenge/net-tcpdump-sniffing`.
   *Alternativamente, passe diretamente como argumento:*
   ```bash
   /challenge/net-tcpdump-sniffing tcpdump_mestre_2026
   ```

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição |
| :--- | :--- |
| `tcpdump -D` | Lista todas as interfaces de rede disponíveis para captura. |
| `tcpdump -i any -c 10` | Captura apenas 10 pacotes em qualquer interface e encerra automaticamente. |
| `tcpdump -r dump.pcap` | Lê e processa um arquivo de captura PCAP previamente salvo. |
| `tshark -i lo -f "udp port 44444" -x` | Equivalente moderno usando o motor CLI do Wireshark (tshark). |

---

## 5. Aplicação Prática no Mundo Real

* **Investigação Forense e SOC (DFIR):** Durante a contenção de um malware, analistas executam o `tcpdump` para capturar os endereços IP dos servidores de Comando e Controle (C2) e as chaves de criptografia compartilhadas na fase inicial de beaconing.
* **Auditorias de Vazamento de Credenciais:** Testes de invasão internos frequentemente usam sniffing para demonstrar a fragilidade de protocolos corporativos legados que enviam senhas sem SSL/TLS (como POP3, IMAP, Telnet e HTTP básico).
