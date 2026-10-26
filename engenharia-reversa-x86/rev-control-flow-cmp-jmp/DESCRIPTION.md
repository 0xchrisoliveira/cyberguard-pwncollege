# Revertendo Desvios Condicionais (CMP, JZ, JNZ)

## Briefing da Missão
Em operações de análise de malware e quebra de proteções de software (crackmes), os alvos raramente possuem senhas estáticas em texto claro. Em vez de uma string simples, os binários utilizam transformações matemáticas, verificações de integridade criptográfica e instruções de desvio condicional baseadas nas flags do processador x86_64.

Nesta missão, sua tarefa é analisar um trecho desmontado de assembly x86_64, compreender o impacto da instrução `cmp` sobre a Zero Flag (`ZF`) no registrador `rflags`, calcular o valor reverso de entrada que força a execução pelo caminho correto e liberar a flag da Cyberguard.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

A manipulação de fluxo de controle por comparações e saltos é o mecanismo primordial de ramificação (`if/else`, loops, switches) da arquitetura von Neumann.

### A Mecânica Interna da Instrução `CMP`

A instrução assembly:
```assembly
cmp eax, 0x12d
```
executa uma operação interna idêntica à instrução `sub eax, 0x12d` (subtração aritmética), **com uma diferença fundamental**:
* O valor de `eax` **não é modificado**. O resultado é descartado pela ALU (Unidade Lógica e Aritmética).
* As **flags de status** no registrador de 64 bits `RFLAGS` são atualizadas com base no resultado da subtração.

### Tabela de Flags Relevantes em x86_64

| Flag | Nome | Quando é setada para 1? |
| :--- | :--- | :--- |
| **ZF** | **Zero Flag (Bit 6)** | Quando o resultado da operação foi exatamente **ZERO** (ou seja, quando `eax == 0x12d`). |
| **SF** | **Sign Flag (Bit 7)** | Quando o resultado foi **negativo** (o bit mais significativo do resultado é 1). |
| **CF** | **Carry Flag (Bit 0)**| Quando ocorreu empréstimo/transbordo em aritmética **não assinada** (`eax < 0x12d`). |
| **OF** | **Overflow Flag (Bit 11)**| Quando ocorreu transbordo em aritmética **com sinal** (excedeu limites de representação). |

### Como as Instruções de Salto (Jumps) Tomam Decisões

```
                      +-------------------+
                      |   cmp eax, 0x12d  |
                      +-------------------+
                                │
                 Atualiza RFLAGS (ZF, SF, CF, OF)
                                │
                                ▼
                      +-------------------+
                      |   jne 0x401250    |
                      +-------------------+
                                │
             ┌──────────────────┴──────────────────┐
             │                                     │
       Se ZF == 0                            Se ZF == 1
   (eax != 301 - Diferente)              (eax == 301 - Igual)
             │                                     │
             ▼                                     ▼
     [ SALTA PARA O ERRO ]                 [ CONTINUA PARA A FLAG! ]
```

### Análise Matemática Reversa
O desmonte da rotina do desafio apresenta a seguinte lógica:
1. `imul eax, eax, 0x7`: o número fornecido pelo usuário é multiplicado por 7.
2. `cmp eax, 0x12d`: o resultado da multiplicação é comparado com o valor hexadecimal `0x12d`.
3. Conversão de base: `0x12d = (1 * 256) + (2 * 16) + 13 = 256 + 32 + 13 = 301`.
4. Equação: `x * 7 = 301` => `x = 301 / 7 = 43`.
5. Logo, quando o input é `43`, `43 * 7 = 301`, tornando a diferença zero e forçando `ZF = 1`, o que impede o desvio `jne` de saltar para o erro!

---

## Passo a Passo de Resolução

1. Execute o desafio para inspecionar os parâmetros operacionais:
   ```bash
   /challenge/rev-control-flow-cmp-jmp
   ```
2. Caso queira revisar o desmonte das instruções e a tabela de flags:
   ```bash
   /challenge/rev-control-flow-cmp-jmp --help
   ```
3. Realize a operação aritmética inversa:
   - Divida 301 por 7, obtendo `43`.
4. Insira o valor **`43`** no terminal:
   ```bash
   /challenge/rev-control-flow-cmp-jmp 43
   ```
5. O debriefing confirmará a ativação da Zero Flag (ZF=1) e imprimirá a flag.

---

## Dicas e Comandos de Terminal no GDB

Se você estivesse depurando dinamicamente este binário no GDB e não conhecesse a chave:

```bash
# 1. Breakpoint na instrução de comparação ou salto:
(gdb) break *0x401248

# 2. Rodar fornecendo qualquer número falso:
(gdb) run <<< "999"

# 3. Inspecionar o valor da Zero Flag (ZF) no registrador eflags:
(gdb) print/t $eflags

# 4. Forçar a inversão da Zero Flag em runtime (Patch Dinâmico):
(gdb) set $eflags |= (1 << 6)   # Ativa ZF (bit 6)

# 5. Prosseguir com a execução (o salto jne não será executado!):
(gdb) continue
```

---

## Aplicação Prática no Mundo Real

Em análises de ameaças de grupos cibercriminosos, o malware frequentemente inclui rotinas de *Anti-Debugging* e *Anti-VM*: ele verifica se o processo está sob depuração chamando a syscall `ptrace` ou checando chaves de registro do VMware/VirtualBox. Se detectar a VM, ele executa um `cmp` e um `jne` que aborta a execução do malware. Analistas de reversa utilizam exatamente o método de inversão de flags (ou patch binário substituindo `jne` por `nop` ou `je`) para neutralizar a evasão do malware e forçá-lo a revelar seu payload real.
