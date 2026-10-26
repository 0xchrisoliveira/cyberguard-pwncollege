# Orquestração de Processos com Pwntools (Interatividade & IPC)

### 📖 Introdução Teórica

No desenvolvimento avançado de exploração binária (*pwn*) e engenharia reversa, o terminal manual torna-se um gargalo operacional inviável. A maioria dos binários modernos exige precisão cirúrgica de temporização, envio de bytes nulos (`\x00`), caracteres de controle e sincronismo bidirecional de mensagens.

Embora linguagens como Python forneçam bibliotecas nativas como `subprocess` e `socket`, o gerenciamento de fluxos brutos de entrada/saída através de pipes manuais frequentemente acarreta **deadlocks de buffer** (quando o processo pai e o processo filho ficam bloqueados esperando dados um do outro) e truncamento de bytes não-ASCII.

O **Pwntools** é o framework de desenvolvimento de exploits em Python consagrado pela comunidade global de cibersegurança e CTFs. Ele abstrai canais de comunicação locais e remotos sob uma interface uniforme denominada **Tube**, proporcionando controle total sobre streams de execução.

```
+-----------------------------------------------------------+
|                   Script do Operador (pwntools)           |
+-----------------------------------------------------------+
       │ io.recvuntil(b"READY")          ▲ io.recvall()
       ▼                                 │
+-----------------------------------------------------------+
|             Subprocesso: /challenge/script-pwntools       |
+-----------------------------------------------------------+
       │                                 ▲
       ▼                                 │
  [ stdout (FD 1) ]                 [ stdin (FD 0) ]
```

#### Vantagens Críticas do Pwntools
1. **Comunicação Bidirecional sem Deadlock:** Alterna de forma transparente entre leitura delimitada (`recvuntil`) e escrita com quebra de linha (`sendline`).
2. **Manipulação Nativa de Tipos Binários:** Funções de empacotamento (`p64`, `p32`) convertem inteiros para arrays de bytes little-endian em arquiteturas x86 e x86_64.
3. **Transição Suave para Shell Interativa:** A instrução `io.interactive()` transfere o controle de teclado e terminal ao operador no momento exato em que um shell remoto ou payload privilegiado é obtido.

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / Pwn):** Indispensável para disparar exploits de corrupção de memória (*Buffer Overflows, ROP Chains, Format Strings*), onde a precisão de alinhamento de memória não tolera conversões incorretas de codificação (como UTF-8 corrompendo bytes de memória).
* **Segurança Defensiva (Blue Team / AppSec):** Utilizado para construir suítes de testes de regressão de segurança (*Security Harness Testing*). Equipes de AppSec utilizam o pwntools para testar se correções e mitigações em código C/C++ realmente resistem a ataques conhecidos em pipelines de integração contínua (CI/CD).

---

### 💻 Exemplos Práticos e Sintaxe

A biblioteca Pwntools opera através de métodos intuitivos de controle de fluxo:

#### Métodos Fundamentais da Interface Tube
* `p = process("/caminho/binario")`: Executa um binário local como subprocesso.
* `p = remote("10.10.10.1", 1337)`: Conecta a um serviço remoto via socket TCP.
* `p.recvuntil(b"padrao")`: Lê o fluxo de saída até encontrar a sequência de bytes indicada.
* `p.sendline(b"dados")`: Envia os bytes fornecidos seguidos de uma quebra de linha (`\n`).
* `p.recvall()`: Lê todos os dados restantes até que o processo feche a conexão.
* `context.log_level = 'debug'`: Ativa o modo de depuração profunda, exibindo todos os bytes enviados e recebidos em formato hexadecimal no terminal.

#### Snippets Práticos e Saídas Observadas

1. **Script de Orquestração com Pwntools (`/tmp/solve_pwn.py`):**
   ```python
   #!/usr/bin/env python3
   from pwn import *

   # Inicia a interação com o binário
   io = process("/challenge/script-pwntools-interaction")

   # Sincroniza com o banner operacional
   io.recvuntil(b"PWNTOOLS_CHALLENGE_READY")

   # Transmite o token de validação exigido
   io.sendline(b"CYBERGUARD_PWNTOOLS_OK")

   # Captura e exibe o resultado final
   print(io.recvall().decode("utf-8"))
   ```

2. **Resolução Rápida via Shell (Método One-Liner):**
   ```bash
   $ echo "CYBERGUARD_PWNTOOLS_OK" | /challenge/script-pwntools-interaction
   ```

---

### 🎯 Seu Objetivo

Criar um script de automação e exploração utilizando a consagrada biblioteca `pwntools`. Seu objetivo é conectar-se ao programa ou serviço via `process()` ou `remote()`, tratar o fluxo de dados recebidos com funções como `recvuntil()` e responder automaticamente com `sendline()`, garantindo uma comunicação rápida, confiável e programática para extrair a flag.
