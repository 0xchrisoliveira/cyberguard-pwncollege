# Reconhecimento Ativo com Nmap (Varredura de Portas & Detecção de Serviços)

### 📖 Introdução Teórica

Em auditorias de segurança e testes de intrusão, a primeira fase operacional é a de **Reconhecimento Ativo (Active Reconnaissance)**. Seu objetivo primordial é mapear a superfície de ataque de um alvo, identificando quais endereços IP estão ativos, quais portas lógicas (de 1 a 65535) estão acessíveis e quais softwares e versões específicas estão respondendo por trás dessas portas.

O **Nmap (Network Mapper)** é o padrão absoluto da indústria para varreduras de rede. Quando o Nmap sonda um host, ele interpreta as respostas da pilha TCP/IP para categorizar o estado de cada porta:

```
[ Nmap Scanner ] ──( Pacote Probe: SYN )──> [ Alvo: Porta X ]
                                                   │
  ┌─────────────────┬──────────────────────────────┴──────────────────────────────┐
  │ Se Aberta:      │ Se Fechada:                                                 │ Se Filtrada (Firewall):
  ▼                 ▼                                                             ▼
[ Responde SYN-ACK ] [ Responde RST-ACK ]                                           [ Nenhuma Resposta / ICMP Inalcançável ]
  │                 │                                                             │
  ▼                 ▼                                                             ▼
Estado: OPEN      Estado: CLOSED                                                Estado: FILTERED
```

#### Modos de Varredura e Técnicas de Detecção
* **TCP SYN Stealth Scan (`-sS`):** O método de varredura mais popular. O Nmap envia um pacote SYN. Se o alvo responder com SYN-ACK, a porta é considerada `open`, mas o Nmap envia imediatamente um pacote RST em vez de completar a conexão com ACK. Por não concluir o handshake, aplicações que registram apenas conexões completas em log frequentemente não detectam a varredura. Requer privilégios administrativos (`root` ou `CAP_NET_RAW`).
* **TCP Connect Scan (`-sT`):** Utiliza a chamada de sistema `connect()` padrão do sistema operacional, completando integralmente o Three-Way Handshake. Funciona com qualquer usuário comum sem privilégios de root, porém gera registros nos logs da aplicação-alvo.
* **Version Detection (`-sV`):** Após identificar uma porta aberta, o Nmap envia uma série de sondas de aplicação (*probes*) e compara os banners retornados contra uma base de milhares de assinaturas, revelando o software e sua versão exata (ex: `Apache 2.4.41` ou `OpenSSH 8.9`).

#### O Mito da Segurança por Obscuridade
Uma prática comumente adotada de forma equivocada por administradores é alterar as portas padrão de serviços críticos (como mudar o SSH da porta 22 para 2222 ou serviços web da porta 80 para 8085). O Nmap torna essa medida ineficaz: uma varredura metódica localiza qualquer porta aberta em frações de segundo, provando que a verdadeira segurança decorre de controle de acesso rigoroso, criptografia e aplicação ágil de patches de segurança.

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team):** Permite descobrir servidores de homologação esquecidos, serviços de depuração internos expostos e aplicações desatualizadas vulneráveis a exploração direta através de vulnerabilidades públicas conhecidas (CVEs).
* **Segurança Defensiva (Blue Team / Gestão de Superfície de Ataque):** Analistas executam varreduras contínuas para detectar imediatamente a introdução de novos serviços na rede corporativa, servidores Shadow IT configurados sem autorização ou backdoors instalados por invasores.

---

### 💻 Exemplos Práticos e Sintaxe

A execução metódica de varreduras com Nmap exige a combinação adequada de parâmetros:

#### Parâmetros e Flags Fundamentais
* `-p <portas>`: Especifica o intervalo de portas a serem escaneadas (ex: `-p 8080-8090`, `-p 22,80,443`, `-p-` para todas as 65535 portas).
* `-sS`: Executa o scan SYN stealth (requer privilégios de root).
* `-sT`: Executa o scan via chamada de sistema connect (usuário desprivilegiado).
* `-sV`: Ativa a detecção profunda de versões de serviços.
* `-sC`: Executa o conjunto padrão de scripts em linguagem Lua (*Nmap Scripting Engine - NSE*).
* `-T<0-5>`: Define o perfil de temporização da varredura, variando de `-T0` (paranóico/evasivo) até `-T4` (agressivo/rápido) e `-T5` (insano).
* `-Pn`: Trata todos os hosts como ativos (ignora a descoberta inicial via ping ICMP).

#### Snippets Práticos e Saídas Observadas

1. **Varredura em Intervalo Restrito de Portas (`-p 8080-8090`):**
   ```bash
   $ nmap -p 8080-8090 127.0.0.1
   Starting Nmap 7.80 ( https://nmap.org )
   Nmap scan report for localhost (127.0.0.1)
   Host is up (0.00012s latency).

   PORT     STATE  SERVICE
   8080/tcp closed http-proxy
   8081/tcp closed blackice-icecap
   8082/tcp closed blackice-alerts
   8083/tcp closed us-srv
   8084/tcp closed fn-val
   8085/tcp open   unknown
   8086/tcp closed d-s-n
   8087/tcp closed simplify
   8088/tcp closed radan-http
   8089/tcp closed unknown
   8090/tcp closed opsmessaging
   ```
   *Identificou-se imediatamente que apenas a porta `8085` está em estado `open`.*

2. **Varredura com Detecção de Versão na Porta Localizada:**
   ```bash
   $ nmap -sV -p 8085 127.0.0.1
   ```

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Para localizar o serviço escondido entre as portas 8080 e 8090 e capturar a flag:

1. **Inicie o daemon alvo do desafio:**
   O binário `/challenge/net-nmap-recon` vincula um serviço em uma porta aleatória dentro do intervalo 8080-8090. Execute-o em segundo plano:
   ```bash
   /challenge/net-nmap-recon &
   ```
   *O programa notificará:*
   ```text
   [*] Um servico foi iniciado em 127.0.0.1 entre as portas 8080 e 8090.
   [*] Faca um scan com 'nmap -p 8080-8090 127.0.0.1' e conecte-se a porta correta.
   ```

2. **Execute a varredura com o Nmap:**
   Sonde o intervalo de portas especificado na interface local:
   ```bash
   nmap -p 8080-8090 127.0.0.1
   ```

3. **Identifique a porta aberta:**
   Na tabela de resultados do Nmap, localize a linha cujo estado seja `open`. Você notará que a porta é a `8085/tcp`.

4. **Conecte-se à porta aberta com o Netcat:**
   Dialogue diretamente com o serviço na porta descoberta:
   ```bash
   nc 127.0.0.1 8085
   ```

5. **Capture e submeta a flag:**
   Ao aceitar a conexão na porta correta, o serviço transmitirá a validação e exibirá a flag:
   ```text
   [+] Porta localizada com sucesso! Flag: pwn.college{...}
   ```
   Copie a string no formato `pwn.college{...}` e insira no painel da plataforma para confirmar a conclusão do desafio.
