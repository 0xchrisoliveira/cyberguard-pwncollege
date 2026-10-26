# Identificação e Análise de Hashes

### 📖 Introdução Teórica
A identificação precisa de formatos de hash e algoritmos criptográficos é uma competência fundamental em auditorias de segurança, testes de invasão e investigações de vazamento de dados corporativos.

**A Falácia da Velocidade: Por que MD5 e SHA-256 não devem proteger senhas?**
Algoritmos de hash criptográfico como MD5, SHA-1 e SHA-256 foram projetados com foco em **velocidade máxima**: seu objetivo primordial é verificar a integridade de grandes volumes de dados (múltiplos gigabytes por segundo).
No entanto, essa velocidade torna-se uma falha crítica quando aplicada a senhas:
* Rigs de mineração e estações de ataque equipadas com GPUs modernas (como NVIDIA RTX 4090) computam **mais de 800 bilhões de hashes MD5 e dezenas de bilhões de hashes SHA-256 por segundo**.
* Uma senha comum de 8 caracteres alfanuméricos possui cerca de $62^8 pprox 2,18 	imes 10^{14}$ combinações. Em MD5, essa senha pode ser quebrada em menos de **4 minutos** por força bruta exaustiva!

**O Papel do Salt (Sal Criptográfico):**
Um *Salt* é uma sequência de bytes pseudoaleatórios gerada individualmente para cada usuário no momento de criação ou troca de senha:
* **Sem Salt:** Dois usuários com a mesma senha possuem hashes idênticos. O atacante pode pré-computar tabelas gigantescas de hashes (*Rainbow Tables*) e recuperar milhões de senhas simultaneamente com buscas diretas em disco.
* **Com Salt:** Mesmo que centenas de usuários escolham a mesma senha, cada um terá um hash completamente distinto. O atacante é forçado a atacar cada hash individualmente, neutralizando completamente Rainbow Tables.

**Hashes Lentos e Adaptativos: Bcrypt e Argon2:**
Para combater o poder paralelo de GPUs e ASICs, foram desenvolvidas funções de derivação lenta que introduzem custo computacional configurável (*work factor*) e dureza de memória (*memory-hardness*):
* **Bcrypt:** Baseado na cifra Blowfish, aplica um fator de custo iterativo ($2^{	ext{cost}}$ rounds). Cada validação de login demora propositalmente de 100 a 300 milissegundos na CPU, reduzindo a capacidade de teste de uma GPU de bilhões para poucos milhares de tentativas por segundo.
* **Argon2 (Argon2id - RFC 9106):** Vencedor da *Password Hashing Competition*, exige o preenchimento de megabytes de memória RAM a cada cálculo de hash, inviabilizando ataques paralelos acelerados por hardware.

**Anatomia Modular de um Hash Bcrypt:**
```
$2b$12$e8uq5e5R6YFh3oQ1K2WkQ.M7Gf1y/8k6E9n0o1p2q3r4s5t6u7v8w
 │   │  └──────────┬─────────┘└──────────────┬──────────────┘
 │   │             │                         └─ Resumo Criptográfico (31 chars)
 │   │             └─ Salt de 128 bits codificado em Base64 modificado (22 chars)
 │   └─ Fator de Custo (Work Factor: 2^12 = 4096 iterações)
 └─ Identificador do Algoritmo ($2b$ = Bcrypt moderno)
```

### 💻 Exemplos Práticos e Sintaxe
Ferramentas de linha de comando para identificação e análise de hashes:

1. **Identificação Automática com o utilitário `hashid` no Linux:**
   ```bash
   hashid -m "e10adc3949ba59abbe56e057f20f883e"
   ```
   *Saída esperada:*
   ```text
   Analyzing 'e10adc3949ba59abbe56e057f20f883e'
   [+] MD5 [Hashcat Mode: 0]
   [+] NTLM [Hashcat Mode: 1000]
   ```

2. **Diferenciação por Padrão Estrutural e Tamanho:**
   * **MD5:** Exatamente 32 caracteres hexadecimais (`0-9`, `a-f`).
   * **SHA-1:** Exatamente 40 caracteres hexadecimais.
   * **SHA-256:** Exatamente 64 caracteres hexadecimais.
   * **NTLM:** 32 caracteres hexadecimais (usado no Windows Active Directory).
   * **Bcrypt:** 60 caracteres iniciando com `$2a$`, `$2b$` ou `$2y$`.
   * **Argon2:** Inicia com `$argon2id$`, `$argon2i$` ou `$argon2d$`.

3. **Gerando e Verificando Hashes Bcrypt em Python:**
   ```python
   import bcrypt

   senha = b"Cyberguard@2026!"
   # Gera salt aleatório e aplica work factor 12:
   hash_bcrypt = bcrypt.hashpw(senha, bcrypt.gensalt(rounds=12))
   print(f"Hash gerado: {hash_bcrypt.decode()}")

   # Validação segura resistente a ataques de temporização:
   confere = bcrypt.checkpw(senha, hash_bcrypt)
   print("Senha válida?", confere)
   ```

### 🎯 O que fazer para resolver o desafio (Passo a Passo)
1. **Localizar e Iniciar o Binário:**
   No terminal do container, execute o desafio de identificação criptográfica:
   ```bash
   /challenge/crypto-hash-identification
   ```
2. **Analisar a Questão do Laboratório:**
   O programa exibirá os artefatos criptográficos e fará a pergunta técnica:
   "Por que funções de hash lentas e adaptativas com salt (como Bcrypt e Argon2) são obrigatórias para armazenamento de senhas corporativas, em vez de MD5 ou SHA-256?"
   Opções:
   * `[1]` Porque MD5 é uma cifra reversível que usa chave privada
   * `[2]` Porque o cálculo propositalmente lento somado ao Salt exclusivo anula Rainbow Tables e inviabiliza força bruta massiva em GPUs
   * `[3]` Porque Bcrypt comprime senhas com perda de integridade para economizar disco
   * `[4]` Porque funções rápidas sofrem de vulnerabilidade de overflow no buffer do kernel
3. **Submeter a Opção Correta:**
   No prompt `[1/2/3/4]`, digite a opção **2**:
   ```text
   2
   ```
   *(Ou execute diretamente com argumento: `/challenge/crypto-hash-identification 2`).*
4. **Capturar e Submeter a Flag:**
   O binário confirmará a resposta correta, detalhará as diretrizes de armazenamento seguro do NIST e exibirá a flag no formato:
   ```text
   pwn.college{...}
   ```
   Copie a flag e registre a pontuação no portal.
