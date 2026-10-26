# Orquestração de Processos com Pwntools (IPC & Exploit Harnessing)

## 1. Briefing da Missão
Operador, no desenvolvimento avançado de exploração binária (pwn) e engenharia reversa, o terminal interativo manual torna-se um gargalo inaceitável. O utilitário `/challenge/script-pwntools-interaction` emite um sinal de sincronismo via `stdout` e aguarda uma resposta com precisão de protocolo via `stdin`.

Sua missão tática é:
1. Dominar a arquitetura da biblioteca `pwntools` para automação de processos locais (`pwn.process`).
2. Implementar um script Python que aguarde o banner `CYBERGUARD_PWNTOOLS_READY`.
3. Responder com o token de validação `CYBERGUARD_PWNTOOLS_OK` e capturar a flag liberada.

---

## 2. Teoria Aprofundada

### 2.1 Por que o Pwntools é o Padrão da Indústria?
Embora linguagens como Python ofereçam o módulo `subprocess`, a manipulação de fluxos brutos de entrada/saída frequentemente resulta em impasses (*deadlocks* de buffer) e truncamento de bytes. O `pwntools` foi projetado especificamente para:
1. **Comunicação Bidirecional Contínua:** Permite alternar entre ler até um padrão (`recvuntil`) e enviar dados (`sendline`) sem fechar descritores de arquivo.
2. **Suporte a Bytes Brutos e Caracteres Nulos:** Permite enviar sequências com bytes `\x00`, `\xff` e endereços de memória empacotados (`p64()`, `p32()`).
3. **Transição Suave para Shell Interativa:** Com um simples comando `io.interactive()`, o controle do terminal é devolvido ao operador assim que o payload é disparado.

```
+-----------------------------------------------------------+
|                   Script do Operador (pwntools)           |
+-----------------------------------------------------------+
       │ io.recvuntil(b"READY")          ▲ io.recvall()
       ▼                                 │
+-----------------------------------------------------------+
|               Subprocesso: /challenge/script-pwntools     |
+-----------------------------------------------------------+
       │                                 ▲
       ▼                                 │
  [ stdout FD 1 ]                  [ stdin FD 0 ]
```

### 2.2 Tabela de Métodos Críticos do Pwntools

| Método | Sintaxe | O que faz? |
| :--- | :--- | :--- |
| **`process()`** | `p = process('/binario')` | Dispara o binário local em um subprocesso controlado. |
| **`remote()`** | `p = remote('10.10.10.1', 1337)` | Abre um socket TCP contra o alvo remoto. |
| **`recvuntil()`** | `p.recvuntil(b"alvo")` | Lê o stream de saída até encontrar os bytes indicados. |
| **`sendline()`** | `p.sendline(b"payload")` | Envia os bytes seguidos por uma nova linha (`\n`). |
| **`clean()`** | `p.clean()` | Limpa e descarta qualquer resíduo restante no buffer de leitura. |
| **`interactive()`**| `p.interactive()` | Conecta o teclado e o monitor diretamente ao subprocesso. |

---

## 3. Passo a Passo de Resolução

### Método 1: Script em Python com Pwntools (Canônico)
Crie o arquivo `/tmp/solve_pwn.py`:

```python
#!/usr/bin/env python3
from pwn import *

# 1. Iniciar o processo
io = process("/challenge/script-pwntools-interaction")

# 2. Aguardar o sinal de sincronismo
io.recvuntil(b"CYBERGUARD_PWNTOOLS_READY")

# 3. Transmitir o token de confirmação
io.sendline(b"CYBERGUARD_PWNTOOLS_OK")

# 4. Capturar a flag no retorno
resultado = io.recvall().decode("utf-8")
print(resultado)
```

Execute o script:
```bash
python3 /tmp/solve_pwn.py
```

### Método 2: One-Liner Rápido via Shell
Como o protocolo aceita dados via stream `stdin`:
```bash
echo "CYBERGUARD_PWNTOOLS_OK" | /challenge/script-pwntools-interaction
```

---

## 4. Dicas e Comandos de Terminal

| Recurso | Exemplo | Descrição |
| :--- | :--- | :--- |
| `p64(addr)` | `p64(0x00401156)` | Empacota um inteiro de 64 bits em bytes little-endian. |
| `p32(addr)` | `p32(0x080484b6)` | Empacota um inteiro de 32 bits em bytes little-endian. |
| `context.log_level` | `context.log_level = 'debug'` | Exibe todos os bytes enviados e recebidos em tempo real. |
| `cyclic(100)` | `cyclic(100)` | Gera uma sequência de Bruijn para calcular offsets de buffer overflow. |

---

## 5. Aplicação Prática no Mundo Real

* **Exploração Binária (Pwn) e ROP Chains:** Em ataques de corrupção de memória e buffer overflow, o pwntools é indispensável para sincronizar o envio de offsets precisos e encadear gadgets da pilha (*Return-Oriented Programming*).
* **Fuzzing de Aplicações:** Pesquisadores de vulnerabilidades utilizam scripts com pwntools para disparar milhares de mutações de entrada por segundo contra serviços locais, monitorando a ocorrência de falhas de segmentação (*Segmentation Faults / SIGSEGV*).
