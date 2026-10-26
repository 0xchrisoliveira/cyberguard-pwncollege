# Sniffing de Pacotes com Tcpdump (Análise de Tráfego & Filtros BPF)

### 📖 Introdução Teórica

Em condições normais de operação, a pilha de rede do kernel Linux descarta qualquer pacote recebido cuja porta de destino não possua um processo escutando ativamente. Além disso, se o tráfego transitar pela rede local mas não for endereçado ao MAC/IP da máquina, a placa de rede ignora os quadros.

A técnica de **Sniffing de Pacotes (Captura Promíscua de Tráfego)** subverte esse comportamento. Quando um utilitário como o **Tcpdump** é iniciado:
1. Ele cria um socket de camada de enlace bruto (`socket(AF_PACKET, SOCK_RAW, ...)`).
2. O subsistema de rede do kernel copia todos os quadros e datagramas recebidos ou transmitidos pela interface especificada.
3. Esses pacotes são submetidos ao motor do **Berkeley Packet Filter (BPF)**, um interpretador de bytecode ultraotimizado que roda diretamente no kernel space para descartar imediatamente pacotes irrelevantes antes que consumam ciclos de CPU no espaço de usuário.

```
       [ Processo Emissor: net-tcpdump-sniffing ]
                         │ (sendto UDP port 44444)
                         ▼
             [ Interface de Loopback: lo ]
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
      [ Pilha IPv4 Normal ]     [ AF_PACKET Raw Socket ]
                                     │
                                     ▼
                            [ Filtro BPF no Kernel ] (udp port 44444)
                                     │
                                     ▼
                              [ tcpdump -i lo -A ]
                                     │
                                     ▼
                          [ Payload em Texto Claro ]
```

#### A Interface de Loopback (`lo`)
A interface de loopback (`127.0.0.1` / `::1`) é uma interface puramente virtual gerenciada pelo kernel. O tráfego que transita por ela nunca atinge circuitos de rede físicos e é frequentemente utilizado por microsserviços e bancos de dados locais. Por padrão, muitos desenvolvedores supõem que comunicações em loopback são "invisíveis", transmitindo tokens confidenciais sem criptografia.

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / Man-in-the-Middle):** Em redes corporativas, atacantes realizam ataques de ARP Spoofing e posicionam sniffers para capturar credenciais de protocolos legados não criptografados (HTTP, FTP, Telnet, LDAP simples, SNMP v1/v2c).
* **Segurança Defensiva (Blue Team / DFIR / SOC):** O `tcpdump` é a ferramenta de primeira resposta em incidentes de segurança. Em servidores headless sem interface gráfica para rodar o Wireshark, o tcpdump permite isolar exfiltrações de dados, beacons de Command & Control (C2) e ataques de negação de serviço em tempo real.

---

### 💻 Exemplos Práticos e Sintaxe

A captura e filtragem cirúrgica de pacotes com o `tcpdump` baseiam-se em flags e expressões BPF:

#### Parâmetros e Flags Críticas
* `-i <interface>`: Define a interface de escuta. Para tráfego interno entre processos na mesma máquina, use `-i lo`. Para interfaces de rede convencionais, use `-i eth0` ou `-i any`.
* `-A`: Renderiza o payload dos pacotes em caracteres legíveis ASCII na tela (ideal para inspecionar protocolos de texto).
* `-X`: Renderiza os bytes em hexadecimal e caracteres ASCII lado a lado.
* `-nn`: Desativa a resolução de nomes de domínio (DNS) e conversão de números de portas em serviços (ex: não converte porta 80 em `http`). Acelera drasticamente a captura.
* `-v` / `-vv`: Aumenta o nível de verbosidade exibindo TTL, identificação IP e opções TCP.
* `-w <arquivo.pcap>`: Grava os pacotes brutos capturados para abertura posterior no Wireshark.
* `-r <arquivo.pcap>`: Lê e analisa um arquivo de captura previamente salvo.

#### Snippets Práticos e Saídas Observadas

1. **Captura na Interface de Loopback com Filtro BPF na Porta 44444:**
   ```bash
   $ tcpdump -i lo -A -nn udp port 44444
   tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
   listening on lo, link-type EN10MB (Ethernet), snapshot length 262144 bytes
   ```

2. **Inspeção do Pacote Interceptado no Terminal:**
   ```text
   12:34:56.789123 IP 127.0.0.1.54321 > 127.0.0.1.44444: UDP, length 43
   E..G..@.@.V...........4.../.CYBERGUARD_SNIFF_SECRET=tcpdump_mestre_2026
   ```
   *Nota-se claramente a string transmitida no corpo do pacote UDP.*

3. **Expressões BPF Comuns para Incident Response:**
   ```bash
   # Capturar apenas pacotes TCP direcionados à porta 80 vindos de um host específico:
   $ tcpdump -i eth0 -nn "tcp dst port 80 and src host 192.168.1.100"
   ```

---

### 🎯 Seu Objetivo

O sistema está transmitindo pacotes de rede sensíveis através da interface de loopback (`lo`). Seu objetivo é colocar o `tcpdump` em execução para capturar o tráfego em tempo real, aplicar filtros adequados para isolar a comunicação relevante e inspecionar o conteúdo dos pacotes transmitidos em texto claro para extrair a credencial ou token necessário para obter a flag.
