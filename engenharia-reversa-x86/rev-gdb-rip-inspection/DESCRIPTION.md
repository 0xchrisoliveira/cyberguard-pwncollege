# Depuração Dinâmica com GDB: Inspecionando o RIP

### 📖 Introdução Teórica

A Engenharia Reversa Dinâmica estuda o binário **durante sua execução ativa** em um ambiente controlado e instrumentado por um depurador (*debugger*), como o GDB (*GNU Debugger*). Enquanto a análise estática examina o código gravado no disco, a análise dinâmica permite congelar a execução a cada ciclo de instrução, inspecionar a memória RAM em tempo real e monitorar os registradores internos do processador.

Para compreender a execução de programas em nível de máquina e avançar para a exploração binária (*pwn*), é obrigatório dominar o ciclo de funcionamento da CPU e a hierarquia de registradores da arquitetura x86_64.

#### O Ciclo Perpétuo da CPU (Fetch-Decode-Execute)
Todo processador moderno opera em um ciclo contínuo de três etapas:
1. **Busca (*Fetch*):** A CPU lê os bytes da instrução contida no endereço de memória apontado pelo registrador **RIP**.
2. **Decodificação e Incremento (*Decode*):** O processador decodifica o opcode e atualiza automaticamente o RIP para que ele aponte para o início da próxima instrução contígua na memória.
3. **Execução (*Execute*):** A CPU executa a operação (aritmética, lógica ou cópia). Se a instrução for um desvio (`jmp`, `call`, `ret`), o processador sobrescreve o RIP com o novo endereço de destino.

```
       +-------------------------------------------------------+
       |                  CICLO DE EXECUÇÃO DA CPU             |
       +-------------------------------------------------------+
       | 1. FETCH   | Busca instrução no endereço do RIP       |
       | 2. DECODE  | Decodifica e avança RIP = RIP + tamanho  |
       | 3. EXECUTE | Executa. Se salto/call, carrega novo RIP |
       +-------------------------------------------------------+
                                  │
                                  ▼
+-----------------------------------------------------------------------------------------+
|                  MAPA DE REGISTRADORES ESSENCIAIS (x86_64 vs 32-bit)                    |
+-------------------+-------------------+-------------------------------------------------+
| 64 BITS (x86_64)  | 32 BITS (x86)     | FUNÇÃO REAL NA CPU                              |
+-------------------+-------------------+-------------------------------------------------+
| RIP               | EIP               | INSTRUCTION POINTER: Guarda o endereço da       |
|                   |                   | PRÓXIMA instrução que será executada pela CPU.  |
+-------------------+-------------------+-------------------------------------------------+
| RFLAGS            | EFLAGS            | FLAGS DE STATUS: Guarda os resultados de testes |
|                   |                   | e comparações (Zero Flag ZF, Carry Flag CF).   |
+-------------------+-------------------+-------------------------------------------------+
| RSP               | ESP               | STACK POINTER: Aponta para o TOPO da pilha.     |
+-------------------+-------------------+-------------------------------------------------+
| RBP               | EBP               | BASE POINTER: Âncora fixa do Stack Frame atual. |
+-------------------+-------------------+-------------------------------------------------+
| RAX               | EAX               | ACUMULADOR: Valor de retorno de funções/syscall.|
+-------------------+-------------------+-------------------------------------------------+
| RDI, RSI, RDX     | EDI, ESI, EDX     | Passagem dos 3 primeiros argumentos em C (Linux)|
+-------------------+-------------------+-------------------------------------------------+
```

#### A Distinção Vital: RIP vs. RFLAGS
Um erro frequente entre analistas iniciantes é confundir o registrador de instrução (**RIP**) com o registrador de flags (**RFLAGS**):
* **RIP (Instruction Pointer):** É exclusivamente um **ponteiro de endereço de memória de 64 bits** (ex: `0x0000000000401156`). Ele **não guarda resultados lógicos**. Ele é o leme da CPU: quem controla o valor do RIP decide para onde a execução do programa salta!
* **RFLAGS (Flags Register):** É uma coleção de **bits individuais de status (0 ou 1)** que registram se uma operação anterior resultou em zero (Zero Flag - ZF), em número negativo (Sign Flag - SF) ou em transbordo (Carry/Overflow Flags).

---

### 💻 Exemplos Práticos e Sintaxe

O depurador GDB permite inspecionar e manipular o registrador RIP passo a passo:

#### 1. Iniciando o GDB e Configurando Sintaxe Intel
```bash
# Iniciar o depurador com o executável alvo:
gdb ./binario

# Configurar o disassembly para sintaxe Intel (mais limpa que AT&T):
(gdb) set disassembly-flavor intel
```

#### 2. Comandos Essenciais para Inspeção do RIP
* `break *main` (ou `b *main`): Define um ponto de parada (*breakpoint*) no endereço inicial da função `main`.
* `run` (ou `r`): Inicia a execução do programa até interceptar o primeiro breakpoint.
* `info registers rip` (ou `i r rip`): Imprime o valor numérico armazenado no registrador RIP.
* `print/x $rip`: Exibe o endereço do RIP em formato hexadecimal puro.
* `x/i $rip`: Desmonta e exibe a instrução exata que está prestes a ser executada em `$rip`.
* `x/5i $rip`: Desmonta as próximas 5 instruções na memória a partir do RIP atual.
* `stepi` (ou `si`): Avança a execução do processador em exatamente **uma única instrução de máquina**.
* `nexti` (ou `ni`): Avança uma instrução sem entrar dentro de chamadas de função (`call`).

#### 3. Exemplo Prático de Sessão GDB
```text
(gdb) break *main
Breakpoint 1 at 0x401126
(gdb) run
Breakpoint 1, 0x0000000000401126 in main ()
(gdb) print/x $rip
$1 = 0x401126
(gdb) x/i $rip
=> 0x401126 <main>:    push   rbp
(gdb) stepi
0x0000000000401127 in main ()
(gdb) x/i $rip
=> 0x401127 <main+1>:  mov    rbp, rsp
(gdb) print/x $rip
$2 = 0x401127
```
*Note como o valor de `$rip` avançou de `0x401126` para `0x401127` após a execução do opcode de 1 byte `push rbp`.*

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Siga as instruções abaixo para validar a compreensão conceitual do registrador RIP:

1. **Invoque o binário do desafio:**
   No terminal, execute o desafio de arquitetura x86_64:
   ```bash
   /challenge/rev-gdb-rip-inspection
   ```

2. **Revise a aula teórica (Opcional):**
   Caso deseje visualizar a dissecação completa dos registradores da CPU:
   ```bash
   /challenge/rev-gdb-rip-inspection --help
   ```

3. **Analise a questão apresentada:**
   O desafio exibirá a seguinte pergunta:
   > *"Em processadores de arquitetura x86 e x86_64, qual é a função exata do registrador EIP / RIP?"*
   * 1) Armazenar o resultado de operações lógicas e de comparação (Zero Flag)
   * **2) Armazenar o endereço de memória da PRÓXIMA instrução a ser executada pela CPU**
   * 3) Armazenar o valor retornado pela última chamada de função executada
   * 4) Armazenar o endereço fixo da base do quadro de pilha (Stack Frame)

4. **Responda com a opção correta:**
   Digite **`2`** no prompt interativo e pressione Enter.
   *Alternativamente, envie diretamente como argumento:*
   ```bash
   /challenge/rev-gdb-rip-inspection 2
   ```

5. **Capture a flag:**
   O sistema confirmará a consolidação do conceito de ponteiro de instrução e imprimirá a flag:
   ```text
   [+] DEBRIEFING TÉCNICO: CONCEITO DE RIP CONSOLIDADO COM PERFEIÇÃO!
   [FLAG DE VALIDAÇÃO CYBERGUARD]:
   pwn.college{...}
   ```
   Copie a flag no formato `pwn.college{...}` e submeta-a para registrar sua pontuação.
