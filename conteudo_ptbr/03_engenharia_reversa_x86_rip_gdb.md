# Guia 03 — Engenharia Reversa: Arquitetura x86_64, RIP e Análise Binária
**Ciclo de Capacitação 2026.2 — Cyberguard Hub & Dojo pwn.college**  
**Classificação:** Educacional / Técnico  
**Público-Alvo:** Integrantes da Cyberguard (Ênfase no Módulo de Engenharia Reversa e Baixo Nível)

---

## 1. O Alerta Diagnóstico: 100% de Erro no Registrador RIP

No diagnóstico inicial da Cyberguard, a questão referente à função do registrador **EIP / RIP** revelou a maior lacuna técnica da equipe: **0 acertos em 8 membros (100% de erro)**. Sete integrantes optaram por "Não sei" e um confundiu sua função com a do registrador `EFLAGS`.

Além disso, **87,5% da equipe errou a distinção entre Análise Estática e Análise Dinâmica**.

Este guia é a pedra fundamental para que a Cyberguard supere essa barreira conceitual e avance com segurança nos desafios práticos do **pwn.college**.

---

## 2. A Arquitetura de Processadores x86 e x86_64

Um processador executa instruções em código de máquina a partir da memória. Para operar em altíssima velocidade, a CPU possui pequenas unidades de armazenamento ultra-rápidas internas chamadas **Registradores**.

No padrão Intel/AMD moderno de 64 bits (x86_64), os registradores possuem 64 bits de largura (prefixo `R`), mas mantêm compatibilidade com suas versões legadas de 32 bits (prefixo `E`) e 16/8 bits:

```
64 bits: |==================== RAX ====================|
32 bits:                     |========== EAX ==========|
16 bits:                                 |==== AX =====|
 8 bits:                                 |==AH==|==AL==|
```

### 2.1 Registradores de Uso Geral (GPR - General Purpose Registers)

Na convenção padrão do Linux (System V AMD64 ABI):

| Registrador (64-bit) | Versão 32-bit | Papel Típico em Execução / ABI |
| :--- | :--- | :--- |
| **RAX** | `EAX` | Acumulador; armazena o **valor de retorno** de funções |
| **RBX** | `EBX` | Registrador de base preservado entre chamadas (*callee-saved*) |
| **RCX** | `ECX` | Contador para loops e operações com strings |
| **RDX** | `EDX` | Armazena dados aritméticos (multiplicação/divisão) |
| **RSI** | `ESI` | *Source Index*; 2º argumento passado para funções |
| **RDI** | `EDI` | *Destination Index*; **1º argumento** passado para funções |
| **RBP** | `EBP` | **Base Pointer / Frame Pointer**: marca a base da pilha atual |
| **RSP** | `ESP` | **Stack Pointer**: aponta sempre para o **topo** da pilha |
| **R8 a R15** | `R8D - R15D`| Registradores adicionais introduzidos na arquitetura 64 bits |

> **Atenção à Passagem de Argumentos em x86_64 Linux:**  
> Os primeiros 6 argumentos inteiros ou ponteiros de uma função são passados nos registradores:  
> `1º: RDI` → `2º: RSI` → `3º: RDX` → `4º: RCX` → `5º: R8` → `6º: R9`. Demais argumentos vão para a pilha (*stack*).

---

## 3. O Registrador Mais Importante: RIP / EIP

```
+-----------------------------------------------------------------------------------+
|                        RIP — INSTRUCTION POINTER                                  |
|                                                                                   |
|  FUNÇÃO ÚNICA E SOBERANA:                                                         |
|  Armazena o endereço de memória da PRÓXIMA instrução que a CPU deve executar.      |
+-----------------------------------------------------------------------------------+
```

* Em arquiteturas de 32 bits, ele se chama **EIP** (*Extended Instruction Pointer*).
* Em arquiteturas de 64 bits, ele se chama **RIP** (*Register Instruction Pointer*).

### 3.1 Como a CPU Opera: O Ciclo de Busca e Execução
1. **Fetch (Buscar):** A CPU lê a instrução que está no endereço apontado pelo **RIP**.
2. **Incrementar RIP:** O RIP é atualizado automaticamente para apontar para a próxima instrução sequencial.
3. **Decode & Execute (Decodificar e Executar):** A CPU executa a instrução lida. Se for um salto (`jmp`), chamada (`call`) ou retorno (`ret`), o RIP é alterado para o novo endereço de destino.

### 3.2 O Erro do Diagnóstico: RIP vs EFLAGS

Um integrante da Cyberguard confundiu o RIP com o `EFLAGS` (ou `RFLAGS` em 64 bits). Vamos desfazer essa confusão de uma vez por todas:

| Característica | RIP / EIP | EFLAGS / RFLAGS |
| :--- | :--- | :--- |
| **Natureza** | É um **Ponteiro de Memória** (endereço de 64 ou 32 bits). | É um conjunto de **Flags de Status** (bits individuais: 0 ou 1). |
| **Função** | Dizer **ONDE** está a próxima instrução a rodar. | Informar o **RESULTADO** da última operação matemática/lógica. |
| **Exemplo de Conteúdo**| `0x0000000000401156` (endereço de memória) | `[ZF=1, CF=0, SF=0]` (Zero Flag, Carry Flag, Sign Flag) |

* Quando uma instrução `cmp rax, 0` é executada:
  * Se `rax` for igual a 0, a flag **ZF (Zero Flag)** no registrador `RFLAGS` se torna `1`.
  * Se for diferente, **ZF** se torna `0`.
  * A instrução seguinte `je target` (*jump if equal*) apenas consulta o bit **ZF** para decidir se altera ou não o **RIP** para o endereço `target`.

### 3.3 Por que Controlar o RIP é o "Santo Graal" de um Exploit?

A CPU é uma máquina cega: ela não julga se o código que está executando é seguro, benigno ou malicioso. Ela simplesmente executa qualquer instrução apontada pelo **RIP**.

Se você encontrar uma vulnerabilidade em um software (como um Buffer Overflow) que permita sobrescrever o valor salvo do RIP na pilha:
* Você pode forçar a CPU a desviar o fluxo para onde você desejar.
* Pode fazê-la pular para uma função oculta de desenvolvedor (`win()` ou `get_flag()`).
* Pode apontar para um código malicioso injetado por você na memória (**Shellcode**).
* Pode encadear fragmentos de código legítimo já existentes no binário (**ROP Gadgets**).

**Quem controla o RIP, controla completamente o computador.**

---

## 4. Análise Estática vs Análise Dinâmica

Em auditoria reversa e análise de malware, existem duas grandes abordagens complementares:

```
                     ANÁLISE DE BINÁRIOS
                              |
        +---------------------+---------------------+
        |                                           |
        v                                           v
ANÁLISE ESTÁTICA                            ANÁLISE DINÂMICA
(Inspecionar sem rodar)                     (Executar sob controle)
- Strings, cabeçalhos ELF/PE                - Debugger (GDB, x64dbg)
- Desmontagem (objdump)                     - Tracing (strace, ltrace)
- Descompilação (Ghidra, IDA)               - Sandbox e monitoramento
```

### 4.1 Análise Estática: Examinar sem Executar

Inspeciona-se a anatomia do arquivo binário diretamente no disco, sem carregá-lo no processador.

* **Vantagens:** Totalmente segura (o malware não tem chance de infectar o sistema); fornece visão global da lógica de fluxo do programa.
* **Ferramentas Clássicas no Terminal:**
  ```bash
  # 1. Identificar o formato do arquivo (arquitetura, se possui debug symbols, se é ELF 64-bit):
  file binario_desafio

  # 2. Extrair texto legível (senhas hardcoded, nomes de funções, mensagens de erro):
  strings -n 8 binario_desafio

  # 3. Ler seções do cabeçalho ELF (verificar proteções NX, PIE, seções executáveis):
  readelf -h binario_desafio

  # 4. Desmontar o código de máquina para Assembly:
  objdump -d -M intel binario_desafio
  ```
* **Ferramentas Avançadas:** **Ghidra** (ferramenta aberta da NSA) e **IDA Pro**, que descompilam o Assembly de volta para um pseudocódigo legível em C.

### 4.2 Análise Dinâmica: Monitorar Durante a Execução

Consiste em colocar o binário em execução em um ambiente isolado (sandbox ou debugger) e observar seu comportamento em tempo real.

* **Vantagens:** Revela o conteúdo exato de variáveis em memória, desvia branches condicionais em runtime e vence algoritmos de ofuscação que só se revelam após rodar.
* **Ferramentas de Tracing Rápido:**
  ```bash
  # Rastrear todas as Chamadas de Sistema (Syscalls - abrir arquivos, conexões de rede):
  strace ./binario_desafio

  # Rastrear chamadas a bibliotecas compartilhadas (ex: strcmp, malloc, printf):
  ltrace ./binario_desafio
  ```

---

## 5. Prática com o GDB (GNU Debugger) & GEF

O GDB é a ferramenta dinâmica por excelência em ambientes Linux e no **pwn.college**. A extensão **GEF (GDB Enhanced Features)** adiciona uma interface rica que exibe registradores, pilha e código de forma visual.

```bash
# Iniciar o binário no GDB:
gdb ./binario_desafio

# Comandos essenciais dentro do GDB:
(gdb) checksec               # Exibe proteções ativas (Canary, NX, PIE)
(gdb) disassemble main       # Mostra o assembly da função main
(gdb) break main             # Cria um breakpoint no início da main
(gdb) run                    # Inicia a execução do programa
(gdb) info registers         # Exibe todos os registradores (RAX, RBX, RIP...)
(gdb) print $rip             # Imprime o valor exato apontado pelo Instruction Pointer
(gdb) nexti                  # Executa uma única instrução Assembly (step over)
(gdb) stepi                  # Entra dentro de chamadas de funções (step into)
(gdb) x/16wx $rsp            # Examina 16 palavras em hexadecimal a partir do topo da pilha
```

---

## 6. O Papel dos Packers (ex: UPX) e a Evasão de Análise

Na questão 11 do teste diagnóstico, abordou-se a finalidade de ferramentas chamadas **Packers**.

### Como Funciona um Packer?
Um packer comprime e/ou cifra as seções de código do executável original e anexa um pequeno trecho descompactador chamado **Unpacking Stub**.

```
BINÁRIO ORIGINAL:          BINÁRIO EMPACOTADO COM UPX:
+-------------------+      +-----------------------------------------+
| .text (Assembly)  | ---> | .upx0 (Vazio / Espaço reservado em RAM) |
| .data (Variáveis) |      | .upx1 (Código original COMPRIMIDO)      |
| .rodata (Strings) |      | Stub Desempacotador (Código inicial)    |
+-------------------+      +-----------------------------------------+
```

### O Desafio para a Segurança:
* **Frustração da Análise Estática:** Se você rodar `strings` ou `objdump` em um binário empacotado, não verá nenhuma das funções reais, nem mensagens, nem chaves de API. Apenas o código compacto do stub!
* **A Resposta da Análise Dinâmica:** Quando o binário empacotado é executado, o primeiro código a rodar é o próprio *stub*. Ele descompacta o código original diretamente na memória RAM e salta o RIP para o ponto de entrada verdadeiro (**OEP — Original Entry Point**). O analista pode pausar o debugger após o stub e capturar o binário totalmente legível na memória.

Para desempacotar um binário tratado com o clássico UPX no terminal:
```bash
upx -d malware_empacotado.elf -o binario_desempacotado.elf
```
Com o binário desempacotado, a análise estática e a busca por vulnerabilidades podem prosseguir normalmente.
