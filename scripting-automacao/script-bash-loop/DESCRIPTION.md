# Automação em Bash e Manipulação de Arquivos (Laços & Aritmética)

## 1. Briefing da Missão
Operador, a infraestrutura de telemetria da Cyberguard exige que seus analistas sejam proficientes no processamento e sumarização de dados numéricos em lote diretamente na linha de comando Unix.

Sua missão tática é:
1. Criar um script ou um one-liner em shell Bash para calcular a soma de todos os números inteiros pares no intervalo fechado de `2` a `100`.
2. Empregar comandos nativos de repetição (`for`, `while`) ou pipelines com utilitários como `seq`, `paste`, `bc` ou `awk`.
3. Submeter o valor resultante ao verificador `/challenge/script-bash-loop` e recuperar a flag.

---

## 2. Teoria Aprofundada

### 2.1 Estruturas de Repetição em Bash
O Bourne-Again Shell (Bash) suporta diversos modelos de repetição para automatizar iterações sobre coleções de itens, diretórios e intervalos numéricos:

```bash
# Sintaxe estilo C (ideal para intervalos matemáticos):
for (( i = 2; i <= 100; i += 2 )); do
    (( soma += i ))
done
```

```bash
# Sintaxe baseada em geradores de sequência (seq):
for num in $(seq 2 2 100); do
    soma=$((soma + num))
done
```

### 2.2 O Poder do Pipeline Unix: A Filosofia de Componentes
Em vez de programar scripts extensos em arquivos `.sh`, a filosofia UNIX preconiza o encadeamento de utilitários simples que realizam uma única tarefa com perfeição:

```
[ seq 2 2 100 ]
  (Gera a sequência: 2, 4, 6, ..., 100)
       │
       ▼ (stdout para stdin via pipe)
[ paste -sd+ - ]
  (Une as linhas usando o delimitador '+': "2+4+6+...+100")
       │
       ▼ (stdout para stdin via pipe)
[ bc ]
  (Calcula a expressão matemática: 2550)
```

### 2.3 Decomposição Matemática (Progressão Aritmética)
A sequência dos inteiros pares `(2, 4, 6, ..., 100)` forma uma Progressão Aritmética clássica:
* Primeiro termo: $a_1 = 2$
* Último termo: $a_n = 100$
* Razão: $r = 2$
* Número de termos: $n = \frac{a_n - a_1}{r} + 1 = \frac{100 - 2}{2} + 1 = 50$

A soma $S_n$ é dada por:
$$S_n = \frac{n \cdot (a_1 + a_n)}{2} = \frac{50 \cdot (2 + 100)}{2} = \frac{50 \cdot 102}{2} = 50 \cdot 51 = 2550$$

---

## 3. Passo a Passo de Resolução

1. **Testando o cálculo com o método `seq | paste | bc`:**
   ```bash
   seq 2 2 100 | paste -sd+ - | bc
   # Saída: 2550
   ```

2. **Testando o cálculo com `awk`:**
   ```bash
   awk 'BEGIN {s=0; for(i=2;i<=100;i+=2) s+=i; print s}'
   # Saída: 2550
   ```

3. **Submetendo o resultado diretamente via pipe:**
   ```bash
   echo $(seq 2 2 100 | paste -sd+ - | bc) | /challenge/script-bash-loop
   ```

4. **Ou passando o valor como argumento:**
   ```bash
   /challenge/script-bash-loop 2550
   ```

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição |
| :--- | :--- |
| `seq <inicio> <passo> <fim>` | Gera uma sequência numérica incremental. |
| `paste -sd<delim> -` | Mescla linhas consecutivas em uma única linha separada pelo delimitador. |
| `bc` | Interpretador de calculadora arbitrária (*Basic Calculator*). |
| `$(( expressao ))` | Expansão aritmética nativa da shell Bash (muito rápida para inteiros). |
| `xargs` | Constrói e executa linhas de comando a partir da entrada padrão. |

---

## 5. Aplicação Prática no Mundo Real

* **Triage Forense e Extração de Métricas:** Analistas de segurança frequentemente utilizam Bash loops para extrair a média de requisições por minuto em ataques DDoS ou somar o volume de bytes transferidos por um endereço IP suspeito a partir de arquivos de log.
* **Automação de Varreduras e Brute-Force:** Em testes de intrusão, scripts em Bash com loops iteram sobre listas de subdomínios, executando verificações de DNS (`dig`, `host`) ou testando credenciais padrão em daemons SSH de centenas de roteadores de uma rede corporativa.
