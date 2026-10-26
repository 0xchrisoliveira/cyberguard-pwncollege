# Reconhecimento Ativo com Nmap (Varredura de Portas & Detecção de Serviços)

## 1. Briefing da Missão
Operador, durante uma auditoria interna na infraestrutura da Cyberguard, identificou-se que um serviço de administração de contingência foi publicado em uma porta não padronizada dentro do intervalo restrito entre as portas `8080` e `8090` em `127.0.0.1`.

Sua missão tática é:
1. Executar o daemon alvo `/challenge/net-nmap-recon`.
2. Empregar o `nmap` para escanear com eficiência a faixa especificada e localizar qual porta está em estado `open`.
3. Conectar-se à porta descoberta (via netcat ou varredura de versão) para extrair o payload da flag.

---

## 2. Teoria Aprofundada

### 2.1 A Mecânica do Port Scanning
Quando um scanner de rede analisa um host, ele testa a acessibilidade de portas lógicas (que variam de 1 a 65535). A resposta enviada pelo sistema operacional define o estado da porta:

```
[ Nmap Scanner ] ──( Pacote SYN )──> [ Host Alvo: Porta X ]
                                            │
  ┌─────────────────┬───────────────────────┴───────────────────────┐
  │ Se Aberta:      │ Se Fechada:                                   │ Se Bloqueada (Firewall):
  ▼                 ▼                                               ▼
[ Responde SYN-ACK ] [ Responde RST-ACK ]                     [ Nenhuma resposta / Timeout ]
  │                 │                                               │
  ▼                 ▼                                               ▼
Estado: OPEN      Estado: CLOSED                                  Estado: FILTERED
```

### 2.2 Tabela Comparativa de Scans do Nmap

| Modo de Scan | Flag | Privilégio Exigido | Mecanismo de Funcionamento | Nível de Ruído em Logs |
| :--- | :---: | :---: | :--- | :---: |
| **TCP SYN (Stealth)** | `-sS` | Root / `CAP_NET_RAW` | Envia SYN, recebe SYN-ACK, envia RST | **Baixo** (não conclui conexão) |
| **TCP Connect** | `-sT` | Usuário Comum | Conclui o 3-Way Handshake via syscall `connect()` | **Alto** (gera log na aplicação) |
| **UDP Scan** | `-sU` | Root | Envia datagramas vazios e aguarda erros ICMP | Muito lento (rate limit) |
| **Version Detection** | `-sV` | Qualquer | Envia probes de aplicação e compara banners | Alto (dialoga com serviço) |

### 2.3 O Mito da Segurança por Obscuridade
Muitas organizações alteram portas padrão de serviços críticos (como mudar o SSH da porta 22 para 2222 ou serviços web da 80 para 8085) acreditando que estarão seguras. 

O Nmap demonstra que essa prática é inútil: uma varredura completa leva poucos segundos para mapear todas as portas ativas, tornando a proteção real dependente de controle de acesso (firewalls), autenticação forte e criptografia.

---

## 3. Passo a Passo de Resolução

1. **Inicie o serviço alvo no primeiro terminal:**
   ```bash
   /challenge/net-nmap-recon
   ```
   *O terminal informará que o serviço está aguardando conexões na faixa 8080-8090.*

2. **Abra outro terminal e execute a varredura direcionada com Nmap:**
   ```bash
   nmap -p 8080-8090 127.0.0.1
   ```
   *Saída observada:*
   ```text
   PORT     STATE SERVICE
   8080/tcp closed http-proxy
   8081/tcp closed blackice-icecap
   ...
   8085/tcp open   unknown
   ...
   ```

3. **Conecte-se à porta aberta identificada (`8085`):**
   ```bash
   nc 127.0.0.1 8085
   ```
   *Alternativamente, usando o próprio Nmap com captura de banner e versão:*
   ```bash
   nmap -sV -p 8085 127.0.0.1
   ```

4. **Recebimento da Flag e Debriefing:**
   O serviço transmitirá a confirmação do reconhecimento concluído e a flag autorizada.

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição |
| :--- | :--- |
| `nmap -p- <alvo>` | Varre todas as 65.535 portas TCP possíveis. |
| `nmap -T4 <alvo>` | Acelera a velocidade da varredura utilizando perfis de timing agressivos. |
| `nmap -sC -sV -p 8085 <alvo>` | Executa scripts de auditoria em Lua (NSE) e identifica a versão do serviço. |
| `nmap -O <alvo>` | Tenta identificar o sistema operacional (OS Fingerprinting) através de nuances na pilha TCP. |

---

## 5. Aplicação Prática no Mundo Real

* **Mapeamento de Superfície de Ataque Externa (EASM):** Red Teams e analistas de segurança usam o Nmap para descobrir servidores de homologação ou painéis de administração esquecidos expostos na internet por desenvolvedores.
* **Auditorias de Conformidade de Rede:** Equipes de Blue Team utilizam scans programados para detectar imediatamente qualquer máquina infectada que tenha aberto portas de backdoor ou listeners não autorizados dentro da rede corporativa.
