# Anatomia do Crash e Medição de Offset na Pilha

### 📖 Introdução Teórica

A Exploração Binária (*Pwn*) representa a fronteira mais técnica e profunda da segurança ofensiva de baixo nível. Para além de ferramentas automatizadas de varredura, a exploração de corrupção de memória exige uma compreensão cirúrgica de como a CPU gerencia a memória física e de como vulnerabilidades em código compilado na linguagem C permitem sequestrar o fluxo de execução de um processo.

O ponto de partida absoluto da exploração binária é a compreensão da **Pilha de Execução (*Call Stack*)** e do fenômeno de **Transbordamento de Buffer (*Stack Buffer Overflow*)**.

#### O Paradoxo Geométrico da Pilha de Execução
Na arquitetura x86_64 sob o padrão System V AMD64 ABI (Linux):
* **Crescimento da Pilha:** A pilha cresce para **baixo**, em direção aos endereços de memória mais baixos (`0x7fffffffffff -> 0x000000000000`). Cada instrução `push` decrementa o registrador `RSP` (`rsp = rsp - 8`).
* **Preenchimento de Buffers:** Quando o programa escreve bytes em um array alocado localmente na pilha (usando funções como `gets`, `strcpy`, `scanf` ou `read`), a gravação ocorre sequencialmente para **cima**, em direção aos endereços maiores de memória!

Essa divergência de direções é a causa física do transbordamento: um array local `char buffer[64]` está fisicamente posicionado em memória logo abaixo de estruturas vitais de controle do programa — especificamente o **`Saved RBP`** e o **`Saved RIP`** (endereço de retorno).

```
[ ENDEREÇOS ALTOS DE MEMÓRIA (0x7fffffffe050) ]
        │
        │  [ DADOS DO CHAMADOR (Caller Stack Frame) ]
        │  +------------------------------------+
        │  | SAVED RIP (Endereço de Retorno - 8B)|  <--- ALVO CRÍTICO DO EXPLOIT!
        │  +------------------------------------+
        │  | SAVED RBP (Base Pointer Anterior)  |  <--- Esmagado pelo transbordo
        │  +------------------------------------+
        │  | char buffer[64] (Espaço alocado)   |  <--- Início da escrita do input
        │  +------------------------------------+       ▲
        │  [ RSP aponta para o topo da pilha ]          │ Gravação avança para CIMA
        ▼                                               │ em direção ao Saved RIP!
[ ENDEREÇOS BAIXOS DE MEMÓRIA (0x7fffffffdff0) ]
```

#### A Mecânica da Falha de Segmentação (SIGSEGV)
Quando um programa vulnerável permite que o usuário envie mais bytes do que a capacidade do buffer (ex: 80 bytes para um espaço de 64):
1. **Bytes 0 a 63:** Preenchem integralmente o array local `buffer[64]`.
2. **Bytes 64 a 71 (8 bytes):** Transbordam e esmagam o `Saved RBP` na memória.
3. **Bytes 72 a 79 (8 bytes):** Sobrescrevem o **`Saved RIP`** com os caracteres enviados (ex: `0x4141414141414141`).
4. **Instrução `ret` (*Return*):** Ao término da função, a CPU executa a instrução `ret` (equivalente a `pop rip`). A CPU retira o valor corrompido do topo da pilha e tenta buscar a próxima instrução nesse endereço.
5. **Crash do Processo:** A Unidade de Gerenciamento de Memória (MMU) detecta que `0x4141414141414141` é um endereço inválido e o kernel encerra o processo com o sinal `SIGSEGV` (*Segmentation Fault / Signal 11*).

---

### 💻 Exemplos Práticos e Sintaxe

Na exploração profissional de binários, nunca se "chuta" o tamanho do payload. Utilizam-se padrões matemáticos não repetíveis (Sequências De Bruijn) para calcular a distância exata até o RIP:

#### 1. Medição Científica de Offset com Pwntools
O framework `pwntools` gera padrões cíclicos nos quais qualquer substring de 4 ou 8 bytes é única em toda a extensão:
```bash
# Gerar um padrão cíclico de 100 caracteres:
python3 -c "from pwn import *; print(cyclic(100).decode())" > payload_teste.txt

# Executar o binário sob o GDB redirecionando o padrão gerado:
gdb ./binario
(gdb) run < payload_teste.txt
# O programa trava com falha de segmentação:
# Program received signal SIGSEGV, Segmentation fault.
# 0x0000000000401156 in vuln ()
# Topo da pilha corrompido com: 0x6161616a ('jaaa')

# Calcular o offset exato até o Saved RIP:
python3 -c "from pwn import *; print(cyclic_find(0x6161616a))"
# Retorno exato: 72 (64 bytes de buffer + 8 bytes de Saved RBP)
```

#### 2. Disparando Payloads de Teste via Linha de Comando
```bash
# Teste via pipe de entrada padrão (stdin):
python3 -c "print('A' * 80)" | /challenge/pwn-stack-anatomy-crash

# Teste passando como argumento de linha de comando:
/challenge/pwn-stack-anatomy-crash $(python3 -c "print('A'*80)")
```

#### 3. Saída Típica de Transbordo Detectado
```text
[*] Dados recebidos: 80 bytes.
[!] ALERTA CRÍTICO DE SEGURANÇA: FALHA DE SEGMENTAÇÃO INDUZIDA (SIGSEGV)
[+] BUFFER OVERFLOW DETECTADO!
    Tamanho do buffer alocado: 64 bytes
    Tamanho da entrada enviada: 80 bytes
    Bytes excedentes na pilha: 16 bytes
[+] O array transbordou a memória local, esmagou o Saved RBP e atingiu o Saved RIP!
```

---

### 🎯 Seu Objetivo

Compreender a estrutura do Stack Frame de uma função na arquitetura x86_64. Seu objetivo é fornecer ao programa uma entrada com tamanho superior ao buffer alocado na pilha, provocando um transbordamento de memória intencional (*buffer overflow*) que sobrescreva dados de controle e resulte em uma falha de segmentação (*Segmentation Fault / Crash*), comprovando a vulnerabilidade.
