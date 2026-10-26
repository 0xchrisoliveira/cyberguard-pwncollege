# Rastreamento de Chamadas de Sistema (Análise Dinâmica com Strace)

## 1. Briefing da Missão
Operador, a equipe de resposta a incidentes da Cyberguard recuperou um binário suspeito `/challenge/script-strace-secrets`. Ele não possui código-fonte disponível e encerra sua rotina operacional sem exibir os parâmetros internos que processa.

Sua missão tática é:
1. Compreender o funcionamento das chamadas de sistema (syscalls) no Linux e a mecânica de interceptação do utilitário `strace`.
2. Executar o binário sob o controle do `strace` filtrando chamadas de arquivos (`openat`, `read`).
3. Descobrir qual arquivo secreto é acessado em tempo de execução e interceptar os bytes da chave lida.
4. Submeter a chave capturada e resgatar a flag.

---

## 2. Teoria Aprofundada

### 2.1 A Fronteira entre User Space e Kernel Space
No modelo de segurança dos processadores x86_64 e do kernel Linux, aplicações comuns rodam em anel de privilégio desprivilegiado (**Ring 3 / User Space**). Elas são impedidas pelo hardware de interagir diretamente com periféricos, discos ou barramentos de memória.

Toda vez que um programa precisa abrir um arquivo, ele carrega o número da chamada de sistema no registrador `RAX`, configura os argumentos nos registradores (`RDI`, `RSI`, `RDX`) e executa a instrução assembly `syscall`. O processador transita temporariamente para **Ring 0 (Kernel Space)** para cumprir a solicitação.

```
+-----------------------------------------------------------+
|            Aplicação (Ring 3 / User Space)                |
|            Código Python / C do binário                   |
+-----------------------------------------------------------+
       │
       │ Instrução Assembly: syscall (RAX=257 / openat)
       ▼
+-----------------------------------------------------------+
|          strace (intercepta via chamada ptrace)           |
|          "openat(AT_FDCWD, '/tmp/.secret', O_RDONLY) = 3" |
+-----------------------------------------------------------+
       │
       ▼
+-----------------------------------------------------------+
|            Kernel Linux (Ring 0 / Supervisor)             |
|            Acesso real ao sistema de arquivos             |
+-----------------------------------------------------------+
```

### 2.2 Tabela de Chamadas de Sistema Críticas em Linux x86_64

| Syscall | Número RAX | Propósito | O que o `strace` revela? |
| :---: | :---: | :--- | :--- |
| **`openat`** | `257` | Abre ou cria arquivos relativo a um diretório | Caminho completo do arquivo, flags de leitura/escrita |
| **`read`** | `0` | Lê bytes a partir de um File Descriptor | Buffer de dados lidos (senhas, chaves, arquivos de config) |
| **`write`** | `1` | Grava bytes em um File Descriptor | Dados enviados para o monitor, disco ou socket |
| **`connect`**| `42` | Inicia conexão de rede TCP contra um IP/porta | Endereço IP do servidor de Comando e Controle (C2) |
| **`execve`** | `59` | Substitui a imagem do processo por outro executável | Linha de comando com que novos processos foram criados |

### 2.3 Raio-X do Diagnóstico da Cyberguard (Questão Q08)
No relatório de desempenho do Ciclo 2026.2, **87,5% da equipe cometeu equívocos na distinção entre análise estática e análise dinâmica**:
* **Análise Estática (`strings`, `objdump`, `ghidra`):** Examina a estrutura do binário sem executá-lo. É ineficaz se o binário decifrar segredos em memória ou estiver ofuscado.
* **Análise Dinâmica (`strace`, `gdb`, `ltrace`):** Observa o binário durante a execução. O binário não pode mentir: para ler a chave no disco, ele é obrigado a pedir ao Kernel via `openat`/`read`, momento em que o `strace` expõe tudo!

---

## 3. Passo a Passo de Resolução

1. **Tentativa inicial direta:**
   ```bash
   /challenge/script-strace-secrets
   ```
   *O binário solicita a chave interceptada durante a execução.*

2. **Execução instrumentada com o `strace`:**
   Execute o binário filtrando chamadas de abertura e leitura de arquivos, garantindo que o buffer não seja truncado com `-s 256`:
   ```bash
   strace -s 256 -e trace=openat,read /challenge/script-strace-secrets
   ```

3. **Análise das linhas emitidas pelo `strace`:**
   Dentre as saídas do interpretador Python, localize a chamada específica do arquivo secreto:
   ```text
   openat(AT_FDCWD, "/tmp/.cyberguard_strace_secret.key", O_RDONLY) = 3
   read(3, "strace_sys_secret_778", 256) = 21
   ```

4. **Submissão do segredo interceptado:**
   Digite a chave `strace_sys_secret_778` no prompt ou execute:
   ```bash
   /challenge/script-strace-secrets strace_sys_secret_778
   ```

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição |
| :--- | :--- |
| `strace -c <comando>` | Gera um sumário estatístico com o número de chamadas e tempo gasto em cada syscall. |
| `strace -p <PID>` | Anexa o sniffer dinâmico a um processo que já está rodando em segundo plano. |
| `strace -f <comando>` | Segue subprocessos filhos criados via `fork` ou `clone`. |
| `ltrace <comando>` | Intercepta chamadas de bibliotecas compartilhadas (libc), como `strcmp` e `malloc`. |

---

## 5. Aplicação Prática no Mundo Real

* **Triage e Análise de Malware em Sandbox:** Quando um artefato desconhecido (ransomware, trojan ou botnet) é recebido pelo SOC, o analista executa o binário sob o controle do `strace` em uma máquina virtual isolada para descobrir quais arquivos ele tenta criptografar e quais IPs ele tenta contactar.
* **Troubleshooting de Aplicações em Produção:** Quando um serviço corporativo falha com a mensagem opaca "Configuration error", o `strace` revela imediatamente qual arquivo de configuração ele tentou abrir e encontrou permissão negada (`EACCES`) ou ausente (`ENOENT`).
