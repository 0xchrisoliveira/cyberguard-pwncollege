# Revertendo Desvios Condicionais (CMP, JZ, JNZ)

### 📖 Introdução Teórica

Em operações reais de engenharia reversa — seja para analisar a lógica de validação de um malware, contornar rotinas de licenciamento (*crackmes*) ou auditar controles de integridade — os programas raramente utilizam senhas puras em texto claro. Em vez disso, a lógica de autorização depende de equações aritméticas seguidas por instruções de **controle de fluxo e desvios condicionais** baseados nas flags do processador x86_64.

#### A Mecânica Interna da Instrução `CMP`
Na arquitetura x86_64, a instrução assembly:
```assembly
cmp destino, origem
```
executa internamente uma operação aritmética de **subtração**:
$$\text{Resultado Temporário} = \text{destino} - \text{origem}$$

**O detalhe crucial de engenharia:**
* O resultado numérico da subtração **é descartado pela Unidade Lógica e Aritmética (ALU)**. Nem o registrador de destino nem o de origem são alterados!
* A **única finalidade da instrução `cmp`** é atualizar os bits de condição no registrador de status **`RFLAGS`**.

#### As Flags de Condição no Registrador `RFLAGS`
O registrador `RFLAGS` de 64 bits contém bits individuais que informam o comportamento da última operação aritmética executada:

| Flag | Nome | Bit | Condição em que é Ativada (`1`) |
| :--- | :--- | :---: | :--- |
| **`ZF`** | **Zero Flag** | 6 | Quando o resultado da operação foi exatamente **ZERO** ($\text{destino} == \text{origem}$). |
| **`SF`** | **Sign Flag** | 7 | Quando o resultado foi **negativo** (bit mais significativo é 1). |
| **`CF`** | **Carry Flag**| 0 | Quando ocorreu transbordo ou empréstimo em aritmética **não assinada** ($\text{destino} < \text{origem}$). |
| **`OF`** | **Overflow Flag** | 11 | Quando ocorreu estouro de capacidade em aritmética **com sinal**. |

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

#### Como os Saltos Condicionais Decidem o Caminho
As instruções de desvio condicional consultam diretamente os bits do `RFLAGS`:
* **`je` / `jz` (*Jump if Equal / Jump if Zero*):** Salta se $\text{ZF} == 1$.
* **`jne` / `jnz` (*Jump if Not Equal / Jump if Not Zero*):** Salta se $\text{ZF} == 0$.
* **`jg` (*Jump if Greater*):** Salta se $\text{ZF} == 0 \text{ e } \text{SF} == \text{OF}$.
* **`jl` (*Jump if Less*):** Salta se $\text{SF} \neq \text{OF}$.

#### Análise Matemática Reversa do Alvo
O trecho de assembly desmontado da rotina deste desafio implementa o seguinte cálculo:
1. `imul eax, eax, 0x7`: O valor numérico fornecido pelo usuário é multiplicado por 7.
2. `cmp eax, 0x12d`: O produto é comparado com o valor hexadecimal `0x12d`.
3. Conversão de base: $0x12d = (1 \times 256) + (2 \times 16) + 13 = 256 + 32 + 13 = 301$.
4. Equação reversa:
$$x \times 7 = 301 \implies x = \frac{301}{7} = 43$$
5. Quando o input é **`43`**, temos $43 \times 7 = 301$. A subtração interna de `cmp` resulta em $301 - 301 = 0$, forçando $\text{ZF} = 1$. O desvio `jne` (*salte se diferente de zero*) avalia $\text{ZF} = 1$, **não executa o salto para o erro** e permite que o programa prossiga até a rotina de entrega da flag!

---

### 💻 Exemplos Práticos e Sintaxe

A dissecação de desvios condicionais e manipulação de flags pode ser realizada via análise algébrica ou através de patching dinâmico de registradores no GDB:

#### 1. Código Assembly Intel Anotado
```assembly
mov    eax, DWORD PTR [rbp-0x4]   ; Carrega o número fornecido pelo usuário
imul   eax, eax, 0x7              ; Multiplica eax por 7
cmp    eax, 0x12d                 ; Compara (eax * 7) com 301 (0x12d)
jne    0x401250 <rotina_bloqueio> ; Se ZF == 0 (diferente), salta para falha!
call   0x401180 <sucesso_flag>    ; Se ZF == 1 (igual), LIBERA A FLAG!
```

#### 2. Bypass Dinâmico de Flags no GDB (Técnica de Engenharia Reversa)
Mesmo sem conhecer o cálculo matemático reverso, um analista com depurador pode forçar a flag no momento exato do teste condicional:
```text
(gdb) break *0x401248             # Breakpoint na instrução jne
(gdb) run
(gdb) info registers rflags        # Inspecionar as flags ativas
(gdb) set $eflags |= (1 << 6)      # Forçar manualmente a Zero Flag (bit 6) para 1!
(gdb) continue                     # Continuar execução: o programa salta para a flag!
```

#### 3. Cálculo Algébrico Rápido no Terminal
```bash
python3 -c "print(0x12d // 7)"
```
*Retorno:* `43`.

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Siga os passos abaixo para resolver a equação assembly e obter a flag:

1. **Inicie o binário do desafio:**
   Execute `/challenge/rev-control-flow-cmp-jmp`:
   ```bash
   /challenge/rev-control-flow-cmp-jmp
   ```

2. **Analise as instruções de máquina desmontadas:**
   O terminal apresentará o trecho assembly de validação:
   ```text
   imul eax, eax, 0x7    ; multiplica entrada por 7
   cmp  eax, 0x12d       ; compara com 301 em decimal
   jne  <rotina_bloqueio>
   ```

3. **Calcule o valor reverso:**
   - Converta `0x12d` para decimal: $1 \times 256 + 2 \times 16 + 13 = 301$.
   - Divida 301 pelo fator multiplicador 7: $\frac{301}{7} = 43$.

4. **Envie o valor calculado:**
   Digite **`43`** no prompt interativo e pressione Enter.
   *Ou execute fornecendo o número diretamente como argumento:*
   ```bash
   /challenge/rev-control-flow-cmp-jmp 43
   ```

5. **Capture a flag:**
   O binário confirmará que $43 \times 7 = 301$, ativando a Zero Flag ($\text{ZF} = 1$) e liberando a flag:
   ```text
   [+] DEBRIEFING TÉCNICO: DESVIO CONDICIONAL RESOLVIDO COM SUCESSO!
   [FLAG DE VALIDAÇÃO CYBERGUARD]:
   pwn.college{...}
   ```
   Copie a string completa `pwn.college{...}` e submeta-a no painel do dojo.
