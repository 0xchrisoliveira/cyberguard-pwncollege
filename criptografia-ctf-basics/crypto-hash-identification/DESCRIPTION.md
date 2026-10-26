# Identificação e Análise de Hashes Criptográficos (Crypto Hash Identification)

## Briefing da Missão
* **Codinome:** Operação Hash Forensics
* **Classificação:** Confidencial / Treinamento de Criptografia & Blue Team
* **Alvo:** Amostras de Hashes Extraídas de Bancos de Dados Vazados
* **Objetivo:** Diferenciar funções de hash ultrarrápidas de integridade (MD5, SHA-1, SHA-256) de funções de derivação lenta com salt (Bcrypt, Argon2id), compreendendo a defesa matemática contra Rainbow Tables e aceleração em GPUs.

Na avaliação diagnóstica 2026.2 da Cyberguard, a **questão 16** apontou que **62,5% dos respondentes não compreendia** a finalidade de hashes lentos com salt para senhas, gerando riscos de implementação incorreta em projetos corporativos. Este desafio consolida os requisitos criptográficos modernos segundo os padrões NIST e RFC 9106.

---

## Teoria Aprofundada

### 1. A Falácia da Velocidade: Por que MD5 e SHA-256 falham em Senhas?
Funções criptográficas tradicionais (MD5, SHA-1, SHA-2) foram arquitetadas com o objetivo de **desempenho máximo**: elas precisam calcular a integridade de arquivos de múltiplos gigabytes em poucos segundos.

No entanto, quando aplicadas a senhas de usuários:
* Uma rig de mineração com 8 placas **NVIDIA RTX 4090** consegue calcular **mais de 800 bilhões de hashes MD5 por segundo**.
* Uma senha comum de 8 caracteres alfanuméricos possui cerca de $62^8 \approx 2,18 \times 10^{14}$ combinações.
* Em MD5, essa senha é quebrada por força bruta em menos de **4 minutos**!

### 2. O Papel do Salt (Sal Criptográfico)
Um **Salt** é uma sequência de bytes pseudoaleatórios gerada individualmente para cada usuário no momento do cadastro:

```
[ Usuário Alice ] -> Senha: "password123" + Salt: "x8A9#kL" -> Hash: 9f8a... (Único)
[ Usuário Bob   ] -> Senha: "password123" + Salt: "mQ2@zP1" -> Hash: 3e7c... (Totalmente Diferente!)
```

* **Sem Salt:** Dois usuários com a mesma senha possuem exatamente o mesmo hash. Um invasor pode pré-computar tabelas gigantescas de senhas comuns (*Rainbow Tables*) e reverter milhões de hashes instantaneamente por busca direta em disco.
* **Com Salt:** Cada usuário exige uma busca independente. O ataque de *Rainbow Tables* torna-se inútil.

### 3. Hashes Lentos e Adaptativos: Bcrypt e Argon2
Para neutralizar a velocidade das GPUs, a indústria de segurança desenvolveu funções lentas baseadas em **Fator de Trabalho (Work Factor)** e **Dureza de Memória (Memory Hardness)**:

```
                               [ SENHA DO USUÁRIO ]
                                        │
           ┌────────────────────────────┴────────────────────────────┐
           ▼                                                         ▼
[ HASH RÁPIDO: MD5 / SHA-256 ]                            [ HASH LENTO: BCRYPT / ARGON2 ]
- Custo computacional fixo e baixo                        - Custo iterativo adaptativo configurável
- 0 uso de memória RAM                                    - Exige megabytes de memória RAM por hash
- Executa em 0,000001 ms                                  - Demora propositalmente 100 a 300 ms
- GPUs testam 100.000.000.000 / seg                       - GPUs testam apenas alguns milhares / seg
- RESULTADO: SENHAS QUEBRADAS EM MINUTOS                  - RESULTADO: FORÇA BRUTA INVIABILIZADA!
```

### 4. Anatomia Estrutural de um Hash Bcrypt
O formato modular padronizado do Bcrypt permite validar a versão e os parâmetros de segurança diretamente pela string:

```
$2b$12$e8uq5e5R6YFh3oQ1K2WkQ.M7Gf1y/8k6E9n0o1p2q3r4s5t6u7v8w
 │   │  └──────────┬─────────┘└──────────────┬──────────────┘
 │   │             │                         │
 │   │             │                         └─ Resumo Criptográfico (31 chars)
 │   │             └─ Salt de 128 bits codificado em Base64 (22 chars)
 │   └─ Fator de Custo (Work Factor: 2^12 = 4096 rounds de Blowfish)
 └─ Identificador do Algoritmo ($2b$ = Bcrypt moderno)
```

---

## Passo a Passo de Resolução

1. Execute o binário do desafio para visualizar a análise dos artefatos:
   ```bash
   /challenge/crypto-hash-identification
   ```
2. Caso deseje consultar a aula teórica completa:
   ```bash
   /challenge/crypto-hash-identification --help
   ```
3. O binário exibirá as opções conceituais sobre o propósito de funções lentas com salt:
   * Opção 1: Cifra reversível com chave privada
   * Opção 2: Cálculo propositalmente lento + Salt que anula Rainbow Tables e força bruta em GPUs
   * Opção 3: Compressão com perda
   * Opção 4: Vulnerabilidade de buffer no kernel
4. Digite a opção correta:
   ```text
   2
   ```
5. Você também pode submeter a resposta diretamente via argumento CLI:
   ```bash
   /challenge/crypto-hash-identification 2
   ```
6. O sistema confirmará a resposta e exibirá a flag de conclusão.

---

## Dicas e Comandos Práticos

* **Identificação Automática de Formatos com `hashid`:**
  ```bash
  # Identificar hashes no terminal Linux:
  hashid -m "e10adc3949ba59abbe56e057f20f883e"
  # Saída: [+] MD5 [Hashcat Mode: 0]
  ```
* **Gerando Hashes Bcrypt em Python com a biblioteca `bcrypt`:**
  ```python
  import bcrypt
  senha = b"minhasenhasuperssegura"
  # Gera salt aleatório com custo 12
  salt = bcrypt.gensalt(rounds=12)
  hash_calculado = bcrypt.hashpw(senha, salt)
  print(hash_calculado)
  # Verificação segura contra timing attacks:
  valido = bcrypt.checkpw(senha, hash_calculado)
  ```
* **Padrão Oficial Recomendado (RFC 9106):**
  O algoritmo vencedor do *Password Hashing Competition* (PHC) é o **Argon2id**, combinando resistência contra ataques de canal lateral (Argon2d) e ataques com memória em GPUs (Argon2i).

---

## Aplicação Prática no Mundo Real
Grandes vazamentos históricos ilustram essa diferença:
* **LinkedIn (2012):** Vaza base de 117 milhões de usuários armazenadas em **SHA-1 puro sem salt**. Em poucos dias, mais de 90% das senhas da rede foram revertidas publicamente.
* **Ashley Madison (2015):** Utilizava Bcrypt com custo 12 para a maioria dos usuários, o que retardou enormemente a reversão das senhas pelos pesquisadores e criminosos, apesar do vazamento massivo dos dados.
