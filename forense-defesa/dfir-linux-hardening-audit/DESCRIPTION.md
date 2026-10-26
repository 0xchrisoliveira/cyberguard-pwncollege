# Hardening de Bastion Host e Serviço SSH

## Briefing da Missão
O Bastion Host de entrada da rede Cyberguard é responsável por isolar a zona desmilitarizada (DMZ) dos servidores internos de produção. Uma auditoria preliminar revelou que o daemon OpenSSH (`sshd`) estava rodando com configurações padrão permissivas, acumulando centenas de conexões rejeitadas por minuto originadas de botnets automatizadas na Internet.

Como analista de segurança defensiva (Blue Team / Hardening), seu objetivo é auditar o arquivo `/tmp/sshd_config_audit`, apontar a configuração remediadora recomendada pelas diretrizes internacionais (CIS Benchmarks) e homologar o isolamento do serviço.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

No diagnóstico do ciclo 2026.2, **50% dos membros demonstraram insegurança quanto ao conceito de hardening e parametrização do SSH**.

### O que é Hardening?
Hardening (endurecimento de sistemas) é a prática proativa de proteção da infraestrutura através da:
1. Eliminação de vetores de ataque desnecessários (fechamento de portas, remoção de pacotes legados como `telnet` e `rsh`).
2. Configuração estrita de controle de acesso seguindo o Princípio do Menor Privilégio (*Least Privilege*).
3. Troca de mecanismos de autenticação fracos por controles criptograficamente robustos.

### As Diretrizes Críticas do `/etc/ssh/sshd_config`

| Diretriz | Valor Inseguro / Padrão | Valor de Hardening (CIS) | Justificativa de Segurança |
| :--- | :--- | :--- | :--- |
| **`PermitRootLogin`** | `yes` | **`no`** | O usuário `root` tem UID 0 e existe em todo Linux. Permitir seu login direto estimula ataques de força bruta contra o superusuário. Com `no`, o atacante deve descobrir um usuário comum e depois escalar privilégios. |
| **`PasswordAuthentication`**| `yes` | **`no`** | Autenticação por senha é suscetível a ataques de dicionário, phishing e vazamentos de credenciais. Apenas chaves criptográficas assimétricas (mínimo RSA 4096 bits ou Ed25519) devem ser autorizadas. |
| **`MaxAuthTries`** | `6` (ou maior) | **`3`** | Limita as tentativas de autenticação antes de encerrar o socket, mitigando softwares de força bruta automatizados. |
| **`X11Forwarding`** | `yes` | **`no`** | Evita que clientes remotos sequestrem sessões de interface gráfica X11 no servidor. |
| **`ClientAliveInterval`** | `0` | **`300`** | Envia mensagens de keepalive e derruba conexões inativas após inatividade prolongada. |

### Diagrama de Fluxo de Autenticação Hardened

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
         │-- Apresenta Chave Privada Ed25519 ---->│
         │                                        ├-- [Authorized_Keys Validado] --> ACESSO CONCEDIDO!
```

---

## Passo a Passo de Resolução

1. Execute o binário do desafio para visualizar a auditoria:
   ```bash
   /challenge/dfir-linux-hardening-audit
   ```
2. Inspecione o arquivo auditado gerado pelo desafio:
   ```bash
   cat /tmp/sshd_config_audit
   ```
3. Identifique as diretrizes inseguras no arquivo:
   - `PermitRootLogin yes`
   - `PasswordAuthentication yes`
4. Avalie as alternativas apresentadas pelo desafio. A alternativa correta implementa `PermitRootLogin no` e `PasswordAuthentication no`.
5. Digite **`2`** no prompt interativo (ou passe `2` como argumento de linha de comando).
6. O debriefing confirmará a aplicação da política CIS e imprimirá a flag.

---

## Dicas e Comandos de Terminal

- **Validando a Sintaxe do SSHD:**
  Sempre que editar `/etc/ssh/sshd_config`, verifique se não há erros de digitação antes de reiniciar o daemon:
  ```bash
  sudo sshd -t
  ```
- **Conferindo a Configuração em Runtime:**
  Para ver todas as diretrizes consolidadas pelo servidor em execução:
  ```bash
  sudo sshd -T | grep -iE "(permitrootlogin|passwordauthentication|maxauthtries)"
  ```
- **Gerando Chaves Ed25519 Fortes:**
  ```bash
  ssh-keygen -t ed25519 -C "operador@cyberguard.local"
  ```

---

## Aplicação Prática no Mundo Real

Quase todos os incidentes de ransomware que afetam servidores Linux começam com força bruta em credenciais fracas de SSH ou logins desprotegidos de root. Ao implementar `PermitRootLogin no` e `PasswordAuthentication no`, todo o volume de tráfego de dicionário automatizado que atinge a porta 22 torna-se inofensivo, forçando o adversário a buscar vetores de ataque muito mais complexos.
