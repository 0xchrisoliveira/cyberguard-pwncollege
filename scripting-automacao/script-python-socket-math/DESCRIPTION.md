# Desafio Matemático em Alta Velocidade (Automação de Processos com Python)

## 1. Briefing da Missão
Operador, a infraestrutura da Cyberguard implementou um mecanismo de integridade anti-automação ingênuo baseado em desafios matemáticos em sequência rápida (*Challenge-Response Protocol*). O sistema exige que você resolva 10 operações de adição consecutivas com um limite rígido de **1.5 segundos por pergunta**.

A velocidade de reflexo e digitação humana é propositalmente incapaz de vencer esse desafio de forma manual. 

Sua missão tática é:
1. Analisar a estrutura do prompt interativo emitido pelo binário `/challenge/script-python-socket-math`.
2. Desenvolver um script de automação em Python (utilizando `pwntools` ou o módulo nativo `subprocess`).
3. Extrair as equações em tempo real com expressões regulares (`re`), calcular o resultado e transmiti-lo de volta antes do estouro do temporizador.
4. Concluir as 10 rodadas e extrair a flag.

---

## 2. Teoria Aprofundada

### 2.1 Latência Humana vs Automação de Máquina
O tempo médio de reação visual humana somado ao tempo de digitação de números de dois dígitos varia entre 1.8 e 3.0 segundos. Quando um sistema impõe limites abaixo de 1.5 segundos em rodadas múltiplas, a probabilidade de falha manual converge para 100%.

```
[ Sistema: Emite Pergunta ] ───( Linha de Texto )───> [ Script Python ]
                                                            │
                                                     [ re.search() Extrai Inteiros ]
                                                            │
                                                     [ Cálculo: a + b (Microssegundos) ]
                                                            │
[ Sistema: Valida em 5ms ]  <──( Envia Resposta )───── [ stdin.write() ]
```

### 2.2 Estratégias de Automação: Subprocess vs Pwntools

| Abordagem | Dependências | Complexidade | Vantagens |
| :--- | :---: | :---: | :--- |
| **`pwntools`** | Requer `pip install pwntools` | Muito Baixa | Sintaxe concisa (`io.recvuntil`, `io.sendline`), controle de buffer perfeito |
| **`subprocess`** | Nenhuma (Biblioteca padrão Python) | Média | Funciona em qualquer ambiente Python minimalista sem internet |
| **`pexpect`** | Requer `pip install pexpect` | Baixa | Simula um pseudo-terminal (PTY) completo |

---

## 3. Passo a Passo de Resolução

### Método 1: Resolução com Pwntools (Recomendado)
Crie um arquivo chamado `/tmp/solve.py`:

```python
#!/usr/bin/env python3
import re
from pwn import *

# Inicializa o processo do desafio
io = process("/challenge/script-python-socket-math")

# Processa as 10 rodadas
for rodada in range(1, 11):
  # Aguarda até o final da pergunta ("? ")
  prompt = io.recvuntil(b"? ").decode("utf-8")
  print(prompt, end="")

  # Extrai os dois operandos da soma usando expressão regular
  match = re.search(r"Quanto e (\d+) \+ (\d+)", prompt)
  if match:
    num1 = int(match.group(1))
    num2 = int(match.group(2))
    resposta = str(num1 + num2)
    print(resposta)
    # Envia a resposta com quebra de linha
    io.sendline(resposta.encode("utf-8"))

# Imprime o debriefing e a flag final
print(io.recvall().decode("utf-8"))
```

Execute o script:
```bash
python3 /tmp/solve.py
```

### Método 2: Resolução com Python Puro (`subprocess`)
Se o ambiente não possuir `pwntools`, utilize `/tmp/solve_native.py`:

```python
#!/usr/bin/env python3
import re
import subprocess

proc = subprocess.Popen(
    ["/challenge/script-python-socket-math"],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True,
    bufsize=1,
)

while True:
  # Ler caractere por caractere ou linha
  output = ""
  while not output.endswith("? "):
    char = proc.stdout.read(1)
    if not char:
      break
    output += char
    print(char, end="", flush=True)

  if not output:
    break

  match = re.search(r"Quanto e (\d+) \+ (\d+)", output)
  if match:
    n1, n2 = int(match.group(1)), int(match.group(2))
    resp = f"{n1 + n2}\n"
    print(resp.strip())
    proc.stdin.write(resp)
    proc.stdin.flush()

# Capturar resto da saída com a flag
resto = proc.stdout.read()
print(resto)
```

---

## 4. Dicas e Comandos de Terminal

| Recurso | Exemplo | Descrição |
| :--- | :--- | :--- |
| `re.search(padrao, texto)` | `re.search(r'\d+', 'ID: 42')` | Localiza a primeira ocorrência do padrão numérico na string. |
| `group(1)` | `match.group(1)` | Retorna o primeiro grupo de captura entre parênteses da regex. |
| `flush=True` | `print("...", flush=True)` | Descarrega o buffer imediatamente sem aguardar quebra de linha. |
| `pwn.process` | `p = process(['cmd', 'arg'])` | Inicia e anexa comunicação bidirecional a um processo local. |

---

## 5. Aplicação Prática no Mundo Real

* **Resolução de Provas de Trabalho (Proof of Work - PoW):** Plataformas e CTFs frequentemente introduzem barreiras criptográficas ou matemáticas de PoW antes de liberar conexões a serviços vulneráveis, exigindo automação em código para prosseguir.
* **Exploração de Race Conditions (Condições de Corrida):** Em vulnerabilidades onde uma brecha de segurança só se abre por microssegundos durante o processamento de transações ou troca de arquivos temporários, scripts automatizados em loop contínuo são o único meio viável de exploração.
