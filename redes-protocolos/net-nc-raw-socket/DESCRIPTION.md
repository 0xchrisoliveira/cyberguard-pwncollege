# Conexões Manuais com Netcat (Sockets TCP Puros & Banner Grabbing)

### 📖 Introdução Teórica

Em redes de computadores baseadas na arquitetura TCP/IP, a maior parte dos usuários e desenvolvedores interage exclusivamente com a **Camada de Aplicação (Camada 7 do modelo OSI)** através de navegadores web (HTTP/HTTPS) ou clientes SSH. Esses protocolos encapsulam dados em cabeçalhos extensos, certificados TLS e regras de formatação rígidas.

No entanto, a comunicação de rede no nível do sistema operacional ocorre na **Camada de Transporte (Camada 4 — TCP ou UDP)** através de abstrações do kernel denominadas **Sockets de Rede**. O utilitário **Netcat (`nc`)** — historicamente consagrado como o "canivete suíço do TCP/IP" — permite interagir diretamente com a Camada 4, transmitindo e recebendo bytes crus através de `stdin` e `stdout` sem qualquer encapsulamento adicional.

```
   [ Operador / Terminal ]
             │ (Bytes puros via stdin / stdout)
             ▼
   [ Netcat (nc) ]
             │ (Criação de socket TCP: socket(AF_INET, SOCK_STREAM))
             ▼
   [ Camada de Transporte (TCP) ]
             │ (Handshake de 3 vias: SYN -> SYN-ACK -> ACK)
             ▼
   [ Conexão Estabelecida na Porta 31337 ]
```

#### O Ciclo de Vida dos Sockets de Rede
No nível de chamadas de sistema (syscalls) do Linux:
1. **Servidor (Listener):** Cria o socket (`socket`), vincula-o a um endereço IP e porta (`bind`), define a fila de espera (`listen`) e bloqueia aguardando conexões (`accept`).
2. **Cliente:** Cria o socket (`socket`) e dispara a conexão contra o IP e porta do servidor (`connect`), iniciando o Three-Way Handshake. Uma vez estabelecido o canal, os dados trafegam via `send()`/`recv()` (ou `read()`/`write()`).

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team):** O Netcat é a ferramenta padrão para o recebimento de conexões reversas (*Reverse Shells*). Ao explorar um alvo vulnerável, o atacante instrui a máquina comprometida a se conectar de volta para um listener netcat (`nc -lvnp 4444`), contornando regras de firewall de entrada (inbound).
* **Segurança Defensiva (Blue Team / SysAdmin):** Permite realizar *Banner Grabbing* e auditorias rápidas em daemons desconhecidos. Analistas testam se portas estão abertas e coletam assinaturas de versões em servidores de email (SMTP porta 25), servidores de terminal ou roteadores sem a sobrecarga de ferramentas pesadas.

---

### 💻 Exemplos Práticos e Sintaxe

O domínio do Netcat exige conhecer seus modos de operação como cliente e como servidor:

#### Comandos Essenciais e Flags
* `nc <IP> <PORTA>`: Conecta como cliente TCP ao endereço e porta indicados.
* `nc -u <IP> <PORTA>`: Conecta utilizando o protocolo UDP em vez de TCP.
* `nc -lvnp <PORTA>`: Inicia um listener local aguardando conexões de entrada:
  * `-l` (*listen*): Coloca o netcat em modo de escuta.
  * `-v` (*verbose*): Exibe detalhes operacionais da conexão.
  * `-n` (*numeric only*): Não resolve nomes via DNS (evita atrasos e vazamentos de tráfego).
  * `-p <porta>` (*port*): Define a porta local para escuta.
* `nc -zv <IP> <PORTAS>`: Modo scan (*zero-I/O*), testa se as portas estão abertas sem enviar dados.
* `ss -tlpn`: Inspeciona todos os processos que estão ouvindo em portas TCP no host local.

#### Snippets de Terminal e Saídas Observadas

1. **Iniciando um Listener de Teste:**
   ```bash
   $ nc -lvnp 4444
   Listening on 0.0.0.0 4444
   ```

2. **Conectando e Realizando Banner Grabbing Manual:**
   ```bash
   $ nc 127.0.0.1 22
   SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.6
   ```
   *O banner do servidor SSH é exibido imediatamente em texto plano.*

3. **Verificação de Portas Locais em Escuta com `ss`:**
   ```bash
   $ ss -tlpn | grep 31337
   LISTEN 0 1 127.0.0.1:31337 0.0.0.0:* users:(("python3",pid=1234,fd=3))
   ```

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Para concluir o desafio e capturar a flag no container pwn.college:

1. **Inicie o serviço do desafio:**
   O binário `/challenge/net-nc-raw-socket` abre um servidor TCP na porta `31337`. Como ele ocupa o terminal aguardando conexões, execute-o em segundo plano (*background*) adicionando um `&` ao final, ou utilize um segundo terminal:
   ```bash
   /challenge/net-nc-raw-socket &
   ```
   *O serviço imprimirá: `[*] Servidor escutando na porta 127.0.0.1:31337...`.*

2. **Conecte-se à porta do serviço utilizando o Netcat:**
   No mesmo terminal (ou no segundo terminal), invoque o `nc`:
   ```bash
   nc 127.0.0.1 31337
   ```

3. **Interaja com o protocolo do servidor:**
   Assim que o canal for aberto, o servidor enviará a mensagem de boas-vindas:
   ```text
   Bem-vindo ao netcat raw socket da Cyberguard!
   Envie 'PING': 
   ```

4. **Transmita a saudação exigida:**
   Digite exatamente `PING` e pressione `Enter`:
   ```text
   PING
   ```

5. **Capture e submeta a flag:**
   O servidor responderá com `PONG!` e emitirá a flag autorizada:
   ```text
   [+] PONG! Flag: pwn.college{...}
   ```
   *Dica avançada: Você também pode resolver o desafio em uma única linha:*
   ```bash
   echo "PING" | nc 127.0.0.1 31337
   ```
   Copie a sequência no padrão `pwn.college{...}` e insira no portal para pontuar.
