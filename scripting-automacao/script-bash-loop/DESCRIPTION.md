# Automação em Bash e Manipulação de Arquivos (Laços & Aritmética)

### 📖 Introdução Teórica

O diferencial de um profissional de elite em segurança ofensiva e defensiva é a capacidade de automatizar tarefas repetitivas diretamente no terminal, sem depender de ferramentas gráficas de terceiros. O **Bourne-Again Shell (Bash)** é a linguagem de controle e automação universal nos sistemas operacionais baseados em Linux.

Através de estruturas de controle de fluxo e repetição (**laços `for` e `while`**), combinadas com a filosofia Unix de encadeamento de pequenos utilitários via pipes (`|`), operadores conseguem processar milhões de registros, realizar cálculos matemáticos complexos e automatizar ataques de força bruta local em frações de segundo.

```
[ Gerador de Sequência: seq 2 2 100 ]
  (Emite: 2, 4, 6, ..., 100 em linhas separadas)
       │
       ▼ (stdout para stdin via pipe)
[ Formatador de Texto: paste -sd+ - ]
  (Mescla as linhas usando o delimitador '+': "2+4+6+...+100")
       │
       ▼ (stdout para stdin via pipe)
[ Calculadora Arbitrária: bc ]
  (Computa a expressão matemática completa: 2550)
```

#### Fundamentação Matemática (Progressão Aritmética)
A soma de todos os inteiros pares de 2 a 100 representa uma Progressão Aritmética (PA) clássica:
* Primeiro termo: $a_1 = 2$
* Último termo: $a_n = 100$
* Razão: $r = 2$
* Quantidade de termos: $n = \frac{a_n - a_1}{r} + 1 = \frac{100 - 2}{2} + 1 = 50$

A soma $S_n$ é dada rigorosamente por:
$$S_{50} = \frac{n \cdot (a_1 + a_n)}{2} = \frac{50 \cdot (2 + 100)}{2} = \frac{50 \cdot 102}{2} = 50 \cdot 51 = 2550$$

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / Brute Force Local):** Laços em Bash são amplamente utilizados para conduzir ataques de força bruta local contra binários protegidos por PIN ou senha (`for pin in {0000..9999}; do ./validador $pin && break; done`), fuzzing rápido de parâmetros de linha de comando e enumeração massiva de arquivos em diretórios restritos.
* **Segurança Defensiva (Blue Team / Triage Forense):** Analistas de resposta a incidentes utilizam laços e pipelines em Bash para sumarizar métricas de tráfego a partir de gigabytes de logs do Apache/Nginx, calcular contagens de requisições por segundo e extrair somas de bytes transferidos por endereços IP maliciosos.

---

### 💻 Exemplos Práticos e Sintaxe

O Bash oferece múltiplas maneiras elegantes de construir repetições e cálculos:

#### Estruturas de Sintaxe no Terminal
1. **Laço Estilo C (Aritmético Nativo):**
   ```bash
   for (( i = 2; i <= 100; i += 2 )); do
       (( soma += i ))
   done
   echo $soma
   ```
2. **One-Liner com Pipelines Unix (`seq` + `paste` + `bc`):**
   ```bash
   seq 2 2 100 | paste -sd+ - | bc
   ```
3. **One-Liner com `awk`:**
   ```bash
   awk 'BEGIN {s=0; for(i=2;i<=100;i+=2) s+=i; print s}'
   ```
4. **Laço de Força Bruta Local com `while`:**
   ```bash
   while read senha; do
       /challenge/autenticador "$senha" 2>/dev/null && break
   done < /usr/share/wordlists/rockyou.txt
   ```

#### Snippets Práticos e Saídas Observadas

* **Executando o Cálculo Direto no Terminal:**
  ```bash
  $ seq 2 2 100 | paste -sd+ - | bc
  2550
  ```

* **Demonstração com Expansão Aritmética do Shell (`$(( ... ))`):**
  ```bash
  $ soma=0; for num in $(seq 2 2 100); do soma=$((soma + num)); done; echo $soma
  2550
  ```

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Para satisfazer a validação do verificador e capturar a flag:

1. **Inspecione a requisição do binário:**
   Execute `/challenge/script-bash-loop` no terminal:
   ```bash
   /challenge/script-bash-loop
   ```
   *O programa solicitará interativamente:*
   ```text
   [?] Digite a soma dos numeros pares de 2 a 100: 
   ```

2. **Calcule a soma via linha de comando:**
   Utilize o pipeline com `seq`, `paste` e `bc` para obter o valor exato:
   ```bash
   seq 2 2 100 | paste -sd+ - | bc
   # Resultado: 2550
   ```

3. **Submeta a resposta calculada:**
   * **Método Interativo:** Digite `2550` no prompt e pressione `Enter`.
   * **Método via Pipeline (Recomendado):**
     ```bash
     echo "2550" | /challenge/script-bash-loop
     ```
   * **Método via Command Substitution:**
     ```bash
     echo $(seq 2 2 100 | paste -sd+ - | bc) | /challenge/script-bash-loop
     ```
   * **Método via Argumento:**
     ```bash
     /challenge/script-bash-loop 2550
     ```

4. **Capture e submeta a flag:**
   Ao receber o valor `2550`, o programa validará a computação e exibirá a flag autorizada:
   ```text
   [+] Calculo correto via Bash!
   pwn.college{...}
   ```
   Copie a string no formato `pwn.college{...}` e insira no portal da Cyberguard.
