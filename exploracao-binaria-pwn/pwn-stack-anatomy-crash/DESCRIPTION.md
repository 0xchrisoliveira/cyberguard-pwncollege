# Anatomia do Crash e Medição de Offset na Pilha

## Briefing da Missão
A divisão de Pesquisa de Vulnerabilidades da Cyberguard localizou um software legado em produção que gerencia sessões de autenticação de operadores. O código-fonte em linguagem C utiliza a função arcaica `gets()` para ler dados do usuário para dentro de um buffer local alocado na pilha de execução (*stack*) de apenas 64 bytes (`char buffer[64]`).

Sua missão como operador ofensivo (Pwn) é investigar os limites físicos desse buffer de memória, construir uma carga útil de entrada (*input payload*) que exceda o tamanho alocado, provocar o transbordamento do espaço de endereçamento da função e induzir uma Falha de Segmentação (*Segmentation Fault / SIGSEGV*), demonstrando o colapso do controle de integridade da CPU.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

No diagnóstico do ciclo 2026.2, **100% dos integrantes da Cyberguard sabiam teoricamente que `strcpy` e `gets` são perigosos** (Questão Q04). Entretanto, ninguém possuía a vivência prática de como a pilha é estruturada fisicamente e como a corrupção de memória se materializa no processador.

### Anatomia da Pilha de Execução (*Stack Frame*)

Na arquitetura x86_64 sob Linux (System V AMD64 ABI):
* **Crescimento da Pilha:** A pilha cresce para **baixo**, em direção aos endereços de memória mais baixos (`0x0000000000000000`).
* **Preenchimento de Buffers:** Quando funções como `strcpy`, `gets` ou `read` escrevem dados em um array, a gravação é feita sequencialmente para **cima**, em direção aos endereços maiores.

```
[ ENDEREÇOS ALTOS DE MEMÓRIA (0x7fffffffe000) ]
        │
        │  [ Argumentos da Função Chamadora ]
        │  [ SAVED RIP ]   <-- Endereço de retorno (8 bytes)  [ALVO DO PWN]
        │  [ SAVED RBP ]   <-- Base Pointer anterior (8 bytes)
        │  [ buffer[64] ]  <-- Array local na pilha (64 bytes)
        │                       │
        │                       ▼ Escrita avança em direção ao Saved RIP!
        │  [ RSP atual ]   <-- Topo da pilha
        ▼
[ ENDEREÇOS BAIXOS DE MEMÓRIA (0x7fffffffd000) ]
```

### O que acontece no Buffer Overflow?

Se o usuário envia exatamente 80 caracteres `"A"` (código ASCII `0x41`):
1. **Bytes 0 a 63:** Ocupam integralmente o espaço legítimo de `buffer[64]`.
2. **Bytes 64 a 71:** Transbordam e sobrescrevem o `Saved RBP` com `0x4141414141414141`.
3. **Bytes 72 a 79:** Sobrescrevem o **`Saved RIP`** com `0x4141414141414141`.
4. **Instrução `ret`:** Ao término da função, a CPU executa `ret` (o equivalente a `pop rip`). A CPU retira `0x4141414141414141` do topo da pilha e tenta buscar a próxima instrução nesse endereço inválido.
5. **Crash:** O hardware de gerenciamento de memória (MMU) detecta violação de acesso e o kernel do Linux finaliza o processo com o sinal `SIGSEGV (Signal 11)`.

---

## Passo a Passo de Resolução

1. Execute o desafio para ler os parâmetros do buffer:
   ```bash
   /challenge/pwn-stack-anatomy-crash
   ```
2. Para estudar a aula didática e entender a anatomia do stack frame:
   ```bash
   /challenge/pwn-stack-anatomy-crash --help
   ```
3. O desafio requer uma entrada superior a 64 bytes para estourar a memória. Crie um payload de 80 bytes utilizando Python e envie via pipe ou argumento:
   ```bash
   python3 -c "print('A' * 80)" | /challenge/pwn-stack-anatomy-crash
   ```
   *Ou passando via argumento:*
   ```bash
   /challenge/pwn-stack-anatomy-crash $(python3 -c "print('A'*80)")
   ```
4. O binário detectará o estouro de buffer, reportará o colapso da pilha e liberará a flag de vitória.

---

## Dicas e Comandos de Terminal

- **Medindo Offsets com Padrões Cíclicos De Bruijn no Pwntools:**
  Não é necessário adivinhar quantos bytes existem até o RIP:
  ```bash
  # Gerar sequência cíclica:
  pwn cyclic 100
  # Descobrir a posição exata de um crash (ex: 0x6161616a):
  pwn cyclic -l 0x6161616a
  ```
- **Inspecionando Core Dumps no Linux:**
  Caso o binário gere um core dump após o SIGSEGV:
  ```bash
  gdb /challenge/pwn-stack-anatomy-crash core
  (gdb) info registers rip
  ```

---

## Aplicação Prática no Mundo Real

Falhas de buffer overflow continuam entre as vulnerabilidades mais críticas da história da computação (como demonstrado nos históricos incidentes do Morris Worm, Code Red e vulnerabilidades críticas em daemons OpenSSL e roteadores domésticos). O domínio da pilha e a medição do offset de crash é o alicerce absoluto sobre o qual se constrói o sequestro de fluxo de execução (*Return-Oriented Programming - ROP*).
