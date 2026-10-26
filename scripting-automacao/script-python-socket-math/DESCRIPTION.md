# Desafio Matemático em Alta Velocidade com Sockets (Automação em Python)

### 📖 Introdução Teórica

Em competições de cibersegurança (CTFs) e auditorias de sistemas no mundo real, desenvolvedores frequentemente implementam mecanismos de mitigação e proteção baseados em **Protocolos de Desafio-Resposta (Challenge-Response)** com restrições temporais severas. O objetivo desses sistemas é barrar interações humanas manuais forçando o interlocutor a provar que possui capacidade de automação programática (*Proof of Work / Automação*).

O tempo médio de reação visual humana somado ao tempo mecânico de digitação em teclado gira em torno de 1.8 a 3.0 segundos. Quando um binário ou serviço de rede impõe um limite estrito de **1.5 segundos por pergunta** ao longo de 10 rodadas consecutivas, a resolução manual torna-se matematicamente inviável.

```
[ Sistema: Emite Pergunta ] ───( Linha de Texto )───> [ Script Python ]
                                                            │
                                                     [ re.search() Extrai Inteiros ]
                                                            │
                                                     [ Cálculo: a + b (Microssegundos) ]
                                                            │
[ Sistema: Valida em 5ms ]  <──( Envia Resposta )───── [ p.sendline() / stdout ]
```

#### Automação de Streams com Python
Para vencer esse tipo de barreira, operadores utilizam scripts em Python para gerenciar canais de comunicação com precisão milimétrica. A interação com processos locais pode ser realizada através da biblioteca padrão `subprocess` ou por meio do framework `pwntools`, combinada com o módulo de expressões regulares (`re`) para parsear e extrair os operandos matemáticos em tempo real.

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / Exploit Automation):** Serviços vulneráveis a ataques de corrupção de memória ou criptografia muitas vezes exigem que o exploit compute respostas dinâmicas em milissegundos antes de disparar o payload principal, contornando mecanismos anti-tampering ou janelas de tempo de *Race Conditions*.
* **Segurança Defensiva (Blue Team / SOAR):** Analistas de automação defensiva (SOAR — *Security Orchestration, Automation and Response*) programam scripts para interagir com APIs de telemetria, validar hashes em dezenas de feeds de inteligência de ameaças simultaneamente e bloquear incidentes em tempo real sem intervenção humana lenta.

---

### 💻 Exemplos Práticos e Sintaxe

A construção do solver automatizado em Python envolve a extração de dados com expressões regulares e a transmissão de respostas via pipes:

#### Ferramentas e Métodos Críticos
* `re.search(r"Quanto e (\d+) \+ (\d+)", texto)`: Extrai os números capturados pelos grupos delimitados por parênteses `(\d+)`.
* `int(m.group(1))` e `int(m.group(2))`: Converte os dígitos extraídos de string para inteiros utilizáveis em operações aritméticas.
* `pwn.process("/challenge/script-python-socket-math")`: Inicia o processo com controle total de seus canais `stdin`, `stdout` e `stderr`.
* `io.recvuntil(b"? ")`: Lê o buffer até o ponto exato onde o programa aguarda a entrada, prevenindo impasses de leitura (*deadlocks*).
* `io.sendline(resposta.encode())`: Envia a resposta calculada seguida pelo caractere de nova linha (`\n`).

#### Snippet Completo de Resolução com Pwntools (`/tmp/solve_math.py`):
```python
#!/usr/bin/env python3
import re
from pwn import *

# Inicia o processo do desafio
io = process("/challenge/script-python-socket-math")

# Processa as 10 rodadas consecutivas
for rodada in range(1, 11):
    # Aguarda o prompt da pergunta
    prompt = io.recvuntil(b"? ").decode("utf-8")
    
    # Extrai os dois inteiros da soma via expressão regular
    match = re.search(r"Quanto e (\d+) \+ (\d+)", prompt)
    if match:
        a = int(match.group(1))
        b = int(match.group(2))
        resultado = str(a + b)
        print(f"[+] Rodada {rodada}: {a} + {b} = {resultado}")
        
        # Envia a resposta calculada
        io.sendline(resultado.encode("utf-8"))

# Imprime o debriefing e a flag final
print(io.recvall().decode("utf-8"))
```

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Para contornar o timeout estrito e capturar a flag:

1. **Teste a execução manual inicial (comprove a impossibilidade):**
   Execute o programa no terminal:
   ```bash
   /challenge/script-python-socket-math
   ```
   *Observe que tentar calcular e digitar a resposta manualmente acarreta a mensagem `[X] Tempo esgotado! Voce precisa automatizar com Python.` após 1.5 segundos.*

2. **Crie o script de automação em `/tmp/solve_math.py`:**
   Utilize o editor de sua preferência ou o utilitário `cat` para salvar o script em `/tmp`:
   ```bash
   cat << 'EOF' > /tmp/solve_math.py
   #!/usr/bin/env python3
   import re
   from pwn import *

   io = process("/challenge/script-python-socket-math")
   for i in range(1, 11):
       prompt = io.recvuntil(b"? ").decode("utf-8")
       match = re.search(r"Quanto e (\d+) \+ (\d+)", prompt)
       if match:
           res = str(int(match.group(1)) + int(match.group(2)))
           io.sendline(res.encode("utf-8"))

   print(io.recvall().decode("utf-8"))
   EOF
   ```

3. **Execute o script de automação:**
   Dispare o resolvedor em Python:
   ```bash
   python3 /tmp/solve_math.py
   ```

4. **Observe a resolução instantânea:**
   O script resolverá as 10 operações matemáticas consecutivas em milissegundos, superando com facilidade o temporizador.

5. **Capture e submeta a flag:**
   Ao final da décima rodada, o programa imprimirá a confirmação e a flag:
   ```text
   [+] Incrivel! Automacao rapida bem-sucedida.
   pwn.college{...}
   ```
   Copie a string no formato `pwn.college{...}` e insira no painel da Cyberguard.
