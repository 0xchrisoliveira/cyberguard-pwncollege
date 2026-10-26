# Descompilação Avançada com Ghidra

## Briefing da Missão
A NSA (National Security Agency) tornou pública a suíte **Ghidra**, um dos ecossistemas de engenharia reversa e descompilação de software mais avançados do mundo. Em investigações de ciberespionagem e análise de exploits desconhecidos (*zero-days*), a descompilação permite transformar milhões de bytes ininteligíveis de código de máquina de volta em funções legíveis na linguagem C.

Sua missão neste desafio é compreender a mecânica por trás da descompilação no Ghidra, dominar a diferença estratégica entre Análise Estática e Análise Dinâmica e consolidar a teoria indispensável para o módulo seguinte de exploração binária.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

No diagnóstico do ciclo 2026.2, **87,5% da equipe errou a diferenciação entre Análise Estática e Análise Dinâmica** (Questão Q08). Esse gargalo impede que o operador saiba escolher a ferramenta certa para cada fase de um engajamento ofensivo ou pericial.

### Matriz Comparativa: Estática vs. Dinâmica

```
+------------------------------------------------------------------------------------------+
| CRITÉRIO            | ANÁLISE ESTÁTICA (Ghidra / IDA)    | ANÁLISE DINÂMICA (GDB / GEF)  |
+---------------------+------------------------------------+-------------------------------+
| O binário roda?     | NÃO. É examinado como dado inerte. | SIM. É executado pela CPU.    |
| Risco de infecção   | ZERO. Seguro para malwares ativos. | ALTO se não estiver isolado.  |
| Visibilidade        | 100% do código (todas as branches).| Apenas o fluxo executado.     |
| Visão da memória    | Estática (endereços relativos).    | Dinâmica (valores reais).     |
| Evasão por malware  | Dificuldade contra packers e XOR.  | Dificuldade contra Anti-Debug.|
+------------------------------------------------------------------------------------------+
```

### Como o Descompilador do Ghidra Funciona?

1. **Disassembly (Desmontagem):** O Ghidra converte os bytes binários em instruções Assembly x86_64.
2. **Geração de P-Code:** Cada instrução assembly é traduzida em uma linguagem intermediária universal chamada P-Code (ex: `mov [rbp-4], eax` vira `STORE (register_space, rbp - 4, eax)`).
3. **Construção do SSA (Static Single Assignment):** O motor matemático analisa o fluxo de vida de cada variável e registrador.
4. **Reconstrução do Pseudocódigo em C:** O Ghidra identifica padrões de loops (`for`, `while`), estruturas condicionais (`if/else`) e chamadas de biblioteca, exibindo o código quase idêntico ao código-fonte original escrito pelo desenvolvedor!

```
Código de Máquina (Bytes)
       │
       ▼
Assembly x86_64 (Disassembly)
       │
       ▼
Linguagem Intermediária (P-Code)
       │
       ▼
Pseudocódigo em C (Decompilação Ghidra)
```

---

## Passo a Passo de Resolução

1. Execute o executável do desafio:
   ```bash
   /challenge/rev-ghidra-decompilation
   ```
2. Caso deseje estudar a teoria detalhada sobre descompilação e boas práticas no Ghidra:
   ```bash
   /challenge/rev-ghidra-decompilation --help
   ```
3. Leia a questão apresentada:
   - Opção 1: Estática executa com parâmetros; dinâmica analisa apenas o assembly (Incorreto - inverte os conceitos).
   - Opção 2: Estática examina o binário e pseudocódigo SEM executá-lo; dinâmica observa o comportamento durante a execução ativa (Correto!).
   - Opção 3: Exclusividades de sistemas operacionais (Incorreto).
   - Opção 4: Ambas executam código (Incorreto).
4. Digite a alternativa **`2`** no terminal (ou envie `2` via parâmetro).
5. O debriefing técnico será emitido e a flag de conclusão do Módulo 8 será liberada!

---

## Dicas e Comandos de Operação no Ghidra

Ao abrir um binário no Ghidra em ambientes reais:
- **Atalho `G` (Go to):** Digite um endereço de memória (ex: `0x00401122`) ou o nome de um símbolo (`main`).
- **Atalho `X` (References to / XREFs):** Revela todos os locais do binário que lêem ou escrevem naquela variável ou função.
- **Atalho `L` (Rename Variable):** Renomeie uma variável descompilada genérica (ex: `iVar1` para `senha_valida`). O Ghidra atualiza automaticamente todo o pseudocódigo!
- **Atalho `T` (Retype Variable):** Altere o tipo de dado de uma variável (ex: de `undefined4` para `int` ou `char*`).

---

## Aplicação Prática no Mundo Real

Na análise de ciberataques patrocinados por Estados-nação (APTs), os artefatos maliciosos têm milhares de linhas de código assembly. Ler assembly instrução por instrução consumiria semanas de trabalho pericial. A descompilação no Ghidra acelera esse processo em mais de 10 vezes, permitindo que os analistas identifiquem em poucos minutos os algoritmos de criptografia de C2, credenciais embutidas e falhas lógicas para neutralização de botnets.
