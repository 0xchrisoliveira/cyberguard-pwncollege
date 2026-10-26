# Hardening de Bastion Host e Serviço SSH

### 📖 Introdução Teórica

Em segurança da informação defensiva, **Hardening** (endurecimento de sistemas) é o processo metódico e proativo de mitigação de vulnerabilidades com o objetivo de reduzir ao máximo a superfície de ataque de um sistema operacional, serviço de rede ou aplicação. A premissa central é que instalações padrão de sistemas operacionais privilegiam compatibilidade e facilidade de uso em detrimento da segurança, deixando portas abertas, protocolos legados ativos e parâmetros excessivamente permissivos.

#### Frameworks e Padrões Internacionais: CIS Benchmarks e Lynis
Para evitar configurações empíricas ou arbitrárias, a indústria adota padrões formais de conformidade como os **CIS Benchmarks** (*Center for Internet Security*), o **NIST SP 800-123** (*Guide to General Server Security*) e ferramentas automatizadas de auditoria como o **Lynis**:
* **Lynis:** Scanner de auditoria de segurança open-source para sistemas Unix/Linux. Ele varre centenas de pontos do sistema (kernel, serviços de rede, regras de firewall, permissões de arquivos, contas de usuário) e gera um índice de conformidade (*Hardening Index*), indicando parâmetros fora das diretrizes CIS.
* **Bastion Host (Jump Server):** Servidor dedicado posicionado na fronteira da rede (DMZ) com a finalidade exclusiva de intermediar e monitorar o acesso administrativo remoto aos servidores internos de produção. Comprometer o Bastion Host concede ao invasor as chaves da rede interna.

```
[ ATACANTE NA INTERNET ]              [ BASTION HOST CYBERGUARD ]
         │                                        │
         │-- Tentativa: login root com senha ---->│
         │                                        ├-- [PermitRootLogin no] --> BLOQUEADO!
         │                                        │
         │-- Tentativa: login user com senha ---->│
         │                                        ├-- [PasswordAuthentication no] --> REJEITADO!
         │                                        │
[ OPERADOR AUTORIZADO ]                           │
         │                                        │
         │-- Apresenta Chave Criptográfica ------>│
         │   (Ed25519 / RSA 4096 bits)            ├-- [authorized_keys Validado] --> SESSÃO CONCEDIDA!
```

#### As Diretrizes Críticas do OpenSSH (`/etc/ssh/sshd_config`)
O serviço SSH (porta TCP 22) é a principal porta de entrada administrativa de servidores Linux na Internet. Se exposto com parâmetros padrão, ele é bombardeado continuamente por botnets automatizadas executando ataques de dicionário e força bruta.

| Parâmetro no `sshd_config` | Valor Padrão / Inseguro | Valor Hardened (CIS) | Justificativa de Segurança |
| :--- | :--- | :--- | :--- |
| **`PermitRootLogin`** | `yes` | **`no`** | O superusuário `root` (UID 0) existe em 100% dos sistemas Linux. Permitir seu login direto elimina a necessidade de o atacante adivinhar o nome de usuário. Com `no`, o invasor deve descobrir um usuário não privilegiado e, posteriormente, encontrar uma falha para escalar privilégios. |
| **`PasswordAuthentication`** | `yes` | **`no`** | Senhas são suscetíveis a ataques de força bruta, phishing e reutilização de credenciais vazadas. A política CIS exige **autenticação estrita por chaves criptográficas assimétricas** (Ed25519 ou RSA 4096 bits). |
| **`MaxAuthTries`** | `6` (ou maior) | **`3`** | Limita as tentativas incorretas de autenticação por conexão TCP antes de desconectar o socket, atrasando severamente varreduras automatizadas. |
| **`X11Forwarding`** | `yes` | **`no`** | Impede o redirecionamento gráfico de janelas X11, evitando que clientes maliciosos espionem ou interceptem sessões gráficas no host. |
| **`ClientAliveInterval`** | `0` (desativado) | **`300`** | Envia probes de keepalive a cada 300 segundos e encerra conexões ociosas abandonadas por administradores. |

---

### 💻 Exemplos Práticos e Sintaxe

A auditoria e o endurecimento do serviço SSH exigem validação metódica de sintaxe e comandos dedicados de inspeção:

#### 1. Auditoria Automatizada do Sistema com Lynis
```bash
# Executar auditoria rápida com o Lynis gerando relatório de conformidade:
lynis audit system --quick

# Auditar especificamente a categoria de autenticação e SSH:
lynis audit --tests-category ssh
```

#### 2. Comandos Operacionais de Manutenção do SSH
Antes de reiniciar o daemon SSH em servidores remotos, é fundamental checar a integridade do arquivo de configuração para evitar a desconexão acidental de todos os operadores (*lockout*):
```bash
# Validar a sintaxe do arquivo de configuração (retorna silencioso se estiver correto):
sudo sshd -t

# Exibir os parâmetros ativos consolidados em tempo de execução pelo daemon:
sudo sshd -T | grep -E "(permitrootlogin|passwordauthentication|maxauthtries)"

# Recarregar as novas regras sem derrubar as sessões ativas dos administradores:
sudo systemctl reload sshd
```

#### 3. Modelo de Configuração Hardened (`/etc/ssh/sshd_config`)
```ini
# Configuração em conformidade com CIS Benchmarks Level 2:
Port 22
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
X11Forwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
```

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Siga os passos abaixo para auditar a configuração e responder corretamente ao desafio:

1. **Inicie o desafio de auditoria:**
   Execute o executável `/challenge/dfir-linux-hardening-audit`:
   ```bash
   /challenge/dfir-linux-hardening-audit
   ```
   *O desafio gerará o arquivo simulado de auditoria em `/tmp/sshd_config_audit`.*

2. **Inspecione o arquivo auditado:**
   Examine o arquivo de configuração gerado para identificar as fragilidades:
   ```bash
   cat /tmp/sshd_config_audit
   ```
   *Observe as diretrizes permissivas:*
   ```text
   PermitRootLogin yes
   PasswordAuthentication yes
   MaxAuthTries 10
   ```

3. **Avalie as opções de remediação:**
   O desafio questionará qual conjunto de diretrizes no `sshd_config` eleva substancialmente a segurança do host e aplica conformidade CIS:
   * 1) `PermitRootLogin yes` e `PasswordAuthentication yes`
   * **2) `PermitRootLogin no` e `PasswordAuthentication no` (exigindo chaves criptográficas)**
   * 3) `MaxAuthTries 100` e `PermitEmptyPasswords yes`
   * 4) `Port 22` sem qualquer restrição de usuário ou chave

4. **Forneça a alternativa correta:**
   Digite **`2`** no prompt interativo e pressione Enter.
   *Alternativamente, envie como argumento de linha de comando:*
   ```bash
   /challenge/dfir-linux-hardening-audit 2
   ```

5. **Capture e registre a flag:**
   O sistema confirmará a validação da política de hardening CIS e imprimirá a flag:
   ```text
   [+] DEBRIEFING TÉCNICO: POLÍTICA DE HARDENING SSH CONFIRMADA!
   [FLAG DE VALIDAÇÃO CYBERGUARD]:
   pwn.college{...}
   ```
   Copie a string completa `pwn.college{...}` e submeta-a para pontuar.
