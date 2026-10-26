# Depuração Dinâmica com GDB: Inspecionando o RIP

## Briefing da Missão
Bem-vindo ao laboratório central de arquitetura de processadores da Cyberguard. Este desafio ataca diretamente a defasagem mais alarmante identificada na avaliação diagnóstica de nivelamento do ciclo 2026.2.

Durante a depuração de binários ELF em código de máquina, compreender os registradores internos da CPU é o divisor de águas entre um script kiddie e um operador de elite. Sua missão é dissecar o ciclo de busca e execução da CPU x86_64, esclarecer sem ambiguidades o papel do registrador **EIP / RIP** e validar seu entendimento perante o depurador dinâmico GDB.

---

## Teoria Aprofundada & O Abismo do Diagnóstico 2026.2

No diagnóstico do ciclo 2026.2:
* **Taxa de Erro:** **100,0%** (0 acertos em 8 membros avaliados).
* **Causa Dominante:** 7 integrantes declararam "Não sei" e 1 confundiu a função de RIP com a do registrador `EFLAGS`.

Essa confusão revela uma falha estrutural no modelo mental de como a CPU executa programas. Vamos desmistificar isso em definitivo.

### O Ciclo Fetch-Decode-Execute e o RIP

Um processador moderno Intel/AMD x86_64 não possui "visão geral" do software. Ele executa uma dança mecânica e cega:

```
+---------------------------------------------------------------------------------+
|                                 CICLO DA CPU                                    |
+---------------------------------------------------------------------------------+
| 1. FETCH (Busca)       | Lê a instrução no endereço de memória apontado por RIP |
| 2. INCREMENTO DE RIP   | RIP aponta automaticamente para a instrução seguinte   |
| 3. DECODE & EXECUTE    | Executa a instrução decodificada (aritmética, salto)   |
+---------------------------------------------------------------------------------+
```

### O Confronto Teórico: RIP vs. RFLAGS

| Característica | RIP / EIP | RFLAGS / EFLAGS |
| :--- | :--- | :--- |
| **Definição** | **Instruction Pointer** (Ponteiro de Instrução) | **Status & Control Flags** (Flags de Estado) |
| **Tipo de Dado** | Ponteiro de 64 bits (ex: `0x0000000000401122`) | Registro de bits individuais (cada bit é 0 ou 1) |
| **Função Exclusiva** | **Aponta para a PRÓXIMA instrução a ser executada** | **Guarda o resultado da última comparação aritmética** |
| **Comportamento em Saltos**| Sobrescrito pelo destino do salto (`jmp`, `call`, `ret`)| Consultado para decidir se o salto ocorre ou não |
| **Flags Típicas** | Não possui flags | **ZF** (Zero Flag), **CF** (Carry), **SF** (Sign), **OF** (Overflow) |

### Os Registradores Críticos da Arquitetura x86_64

```
64-bit       32-bit      Papel Principal
-------------------------------------------------------------------------
RIP          EIP         Ponteiro da PRÓXIMA Instrução (Controle de Fluxo)
RSP          ESP         Stack Pointer (Topo atual da Pilha)
RBP          EBP         Base / Frame Pointer (Base do quadro da função)
RAX          EAX         Acumulador & Valor de Retorno de Funções
RDI          EDI         1º Argumento de Função (System V AMD64 ABI)
RSI          ESI         2º Argumento de Função
RDX          EDX         3º Argumento de Função
RFLAGS       EFLAGS      Flags de Comparação Condicional (ZF, CF, SF, OF)
```

---

## Passo a Passo de Resolução

1. Execute o desafio no terminal:
   ```bash
   /challenge/rev-gdb-rip-inspection
   ```
2. Para revisar a aula didática completa com os comandos de GDB:
   ```bash
   /challenge/rev-gdb-rip-inspection --help
   ```
3. Leia com atenção a pergunta sobre a função do registrador EIP/RIP:
   - Opção 1: Flags de comparação (Incorreto - função do RFLAGS).
   - Opção 2: Endereço de memória da PRÓXIMA instrução a ser executada pela CPU (Correto!).
   - Opção 3: Valor retornado pela função (Incorreto - função do RAX).
   - Opção 4: Endereço fixo da base do quadro de pilha (Incorreto - função do RBP).
4. Informe a alternativa **`2`** no prompt (ou passe `2` como argumento).
5. O debriefing técnico será emitido e a flag de validação será impressa.

---

## Dicas e Comandos de Terminal no GDB

Pratique a inspeção do RIP em tempo real utilizando o depurador GDB:

```bash
# 1. Iniciar o binário sob controle do GDB:
gdb /bin/ls

# 2. Inserir breakpoint no ponto de entrada principal:
(gdb) break *main

# 3. Iniciar a execução até o breakpoint:
(gdb) run

# 4. Inspecionar o registrador RIP e a instrução que ele aponta:
(gdb) info registers rip
(gdb) x/i $rip

# 5. Avançar exatamente UMA instrução de máquina (Step Instruction):
(gdb) stepi

# 6. Observar o RIP incrementado para o novo endereço:
(gdb) print/x $rip
(gdb) x/i $rip
```

---

## Aplicação Prática no Mundo Real

Na exploração de vulnerabilidades de memória (Binary Exploitation / Pwn), o atacante utiliza transbordamentos de buffer (*stack buffer overflow*) para sobrescrever o `Saved RIP` preservado na pilha. Quando a função retorna (`ret`), a CPU faz o equivalente a `pop rip`, carregando o endereço injetado diretamente no registrador de instrução. A partir desse momento, a CPU executa cegamente o payload do invasor. Dominar o RIP é o primeiro passo para criar e mitigar exploits de controle de fluxo de execução.
