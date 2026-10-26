# Rastreamento de Chamadas de Sistema (Análise Dinâmica com Strace)

### 📖 Introdução Teórica

Na arquitetura de segurança dos processadores modernos x86_64 e do kernel Linux, aplicações comuns de usuário executam em um anel de privilégio desprivilegiado conhecido como **Ring 3 (User Space / Espaço do Usuário)**. O hardware impede que tais programas acessem diretamente a memória de outros processos, interajam diretamente com o disco rígido ou enviem pacotes para interfaces físicas de rede.

Sempre que uma aplicação necessita interagir com o mundo externo (abrir um arquivo, ler dados do disco, criar um processo filho ou conectar-se à internet), ela é obrigada a solicitar a intervenção do kernel no **Ring 0 (Kernel Space / Supervisor)** através de uma **Chamada de Sistema (Syscall)**.

```
+-----------------------------------------------------------+
|              Aplicação (Ring 3 / User Space)              |
|              Código executável do binário                 |
+-----------------------------------------------------------+
       │
       │ Instrução Assembly: syscall (RAX = 257 / openat)
       ▼
+-----------------------------------------------------------+
|          strace (intercepta via chamada ptrace)           |
|          "openat(AT_FDCWD, '/tmp/.secret', O_RDONLY) = 3" |
+-----------------------------------------------------------+
       │
       ▼
+-----------------------------------------------------------+
|            Kernel Linux (Ring 0 / Supervisor)             |
|            Acesso real ao hardware e arquivos             |
+-----------------------------------------------------------+
```

#### Como o `strace` Funciona Internamente
O **`strace` (System Call Trace)** utiliza a chamada de sistema `ptrace(PTRACE_SYSCALL, ...)` do Linux. Essa syscall permite que o `strace` anexe um interceptador ao processo-alvo. Cada vez que a CPU encontra a instrução assembly `syscall`, a execução do binário é congelada antes e depois de transitar para o kernel, permitindo ao `strace` decodificar e imprimir na tela os argumentos exatos passados nos registradores (`RDI`, `RSI`, `RDX`, etc.) e o valor retornado em `RAX`.

#### Análise Estática vs. Análise Dinâmica
* **Análise Estática (`strings`, `objdump`, `Ghidra`):** Avalia os bytes do executável em repouso sem dispará-lo. Embora indispensável, é facilmente frustrada por técnicas de ofuscação, empacotadores (*packers*) ou strings cifradas que só são decodificadas em tempo de execução.
* **Análise Dinâmica (`strace`, `ltrace`, `GDB`):** Observa o binário durante a execução. O programa não pode mentir para o kernel: para ler uma chave no disco, ele obrigatoriamente terá que passar o nome do arquivo para `openat()` e carregar os bytes do segredo via `read()`. O `strace` revela essas operações instantaneamente sem exigir engenharia reversa profunda.

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / Engenharia Reversa):** Permite recuperar senhas, caminhos de arquivos de configuração camuflados e chaves criptográficas em segundos, além de identificar falhas lógicas em checagens de validação de arquivos.
* **Segurança Defensiva (Blue Team / Triage de Malware no SOC):** Analistas de segurança executam malwares suspeitos dentro de sandboxes monitoradas por `strace` para descobrir imediatamente quais arquivos estão sendo criptografados por ransomware, quais chaves de persistência estão sendo gravadas e quais endereços IP de Comando & Controle (C2) estão sendo contactados.

---

### 💻 Exemplos Práticos e Sintaxe

O uso cirúrgico do `strace` envolve filtros e controle de buffer de leitura:

#### Parâmetros e Flags Fundamentais
* `-e trace=<syscalls>`: Filtra apenas as chamadas de sistema especificadas (ex: `-e trace=openat,read`, `-e trace=network`, `-e trace=file`).
* `-s <tamanho>`: Define o tamanho máximo de caracteres exibidos para buffers de strings (o padrão é truncar em apenas 32 caracteres; use `-s 256` para ler chaves e tokens inteiros).
* `-f`: Segue processos filhos gerados via chamadas `fork` ou `clone`.
* `-p <PID>`: Conecta o interceptador a um processo que já está rodando em segundo plano.
* `-c`: Modo contagem/sumário; exibe uma tabela estatística consolidando quantas chamadas cada syscall efetuou e o tempo gasto.

#### Tabela de Chamadas de Sistema Críticas em Linux x86_64
| Syscall | Número RAX | Propósito no Kernel | O que o `strace` revela |
| :---: | :---: | :--- | :--- |
| **`openat`** | `257` | Abre ou cria arquivos relativo a um diretório | Caminho completo do arquivo e modo de abertura |
| **`read`** | `0` | Lê bytes a partir de um File Descriptor | Conteúdo bruto de senhas, chaves e arquivos |
| **`write`** | `1` | Grava bytes em um File Descriptor | Mensagens enviadas ao terminal, arquivo ou socket |
| **`connect`**| `42` | Inicia conexão de rede TCP contra um IP/porta | Endereço IP e porta do servidor de destino |
| **`execve`** | `59` | Executa um novo programa | Argumentos e binários derivados |

#### Snippets Práticos e Saídas Observadas

1. **Rastreamento de Chamadas de Abertura e Leitura de Arquivo:**
   ```bash
   $ strace -s 256 -e trace=openat,read /challenge/script-strace-secrets
   ```

2. **Linhas de Saída Típicas Emitidas pelo Kernel:**
   ```text
   openat(AT_FDCWD, "/tmp/.<senha_interceptada_aqui>.key", O_RDONLY) = 3
   read(3, "strace_sys_secret_778", 256) = 21
   ```
   *O `openat` retornou o File Descriptor número 3, e o `read` seguinte leu 21 bytes correspondentes à chave secreta:* `strace_sys_secret_778`.

---

### 🎯 Seu Objetivo

O executável `/challenge/script-strace-secrets` valida uma senha secreta internamente em memória sem jamais exibi-la na tela. Seu objetivo é monitorar a execução do programa em nível de chamadas de sistema utilizando o utilitário `strace`, interceptar as syscalls (como `read` ou `open`) no momento em que o segredo é manipulado pelo kernel e utilizá-lo para solucionar o desafio.
