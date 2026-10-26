# Guia 08 — Criptografia Aplicada & Fundamentos de CTF
**Ciclo de Capacitação 2026.2 — Cyberguard Hub & Dojo pwn.college**  
**Classificação:** Educacional / Técnico  
**Público-Alvo:** Integrantes da Cyberguard (Foco em Criptografia, Armazenamento Seguro e Resolução de Desafios CTF)

---

## 1. Introdução e o Cenário no Diagnóstico

No teste diagnóstico da Cyberguard, a área de Criptografia e Prática de CTF revelou confusões conceituais graves:
* **75% da equipe errou ou não soube identificar uma string clássica em Base64**, chegando a confundi-la com um Hash MD5.
* **62,5% dos respondentes não soube diferenciar funções de hash rápido (MD5/SHA) de algoritmos modernos e lentos de armazenamento seguro de senhas (bcrypt/Argon2)**.

O primeiro passo para o sucesso em competições de CTF e em auditorias de segurança é saber exatamente com que tipo de dado você está lidando.

---

## 2. A Tríade Fundamental: Encoding vs Hashing vs Encryption

Um dos erros mais comuns em novatos na segurança da informação é utilizar os termos "criptografar", "codificar" e "gerar hash" como se fossem sinônimos. **Eles operam sob princípios matemáticos e objetivos completamente diferentes:**

```
+-----------------------------------------------------------------------------------+
| 1. ENCODING (Codificação)                                                         |
| Propósito: Usabilidade e compatibilidade de transporte de dados.                  |
| Segurança: NENHUMA.                                                               |
| Reversibilidade: 100% reversível por qualquer pessoa SEM chave.                   |
| Exemplos: Base64, Hexadecimal (ASCII-Hex), URL Encoding, Morse.                   |
+-----------------------------------------------------------------------------------+
                                         │
+-----------------------------------------------------------------------------------+
| 2. HASHING (Função de Resumo Criptográfico)                                      |
| Propósito: Garantir integridade de arquivos e assinaturas digitais.               |
| Segurança: Unidirecional (One-Way) — Impossível reverter matematicamente.         |
| Saída: Tamanho fixo (ex: SHA-256 gera sempre 256 bits, independentemente da entrada).|
| Exemplos: SHA-256, SHA-3, MD5 (inseguro), bcrypt, Argon2.                         |
+-----------------------------------------------------------------------------------+
                                         │
+-----------------------------------------------------------------------------------+
| 3. ENCRYPTION (Cifragem / Criptografia Real)                                      |
| Propósito: Confidencialidade e sigilo da informação.                              |
| Segurança: Alta, baseada no segredo da chave criptográfica.                       |
| Reversibilidade: 100% reversível APENAS por quem possui a CHAVE correta.         |
| Exemplos: AES (Simétrica), ChaCha20, RSA (Assimétrica), ECC.                      |
+-----------------------------------------------------------------------------------+
```

---

## 3. Dissecando a Questão do Teste: Reconhecimento e Decodificação de Base64

Na questão 20 do teste diagnóstico, foi apresentada uma string misteriosa com a instrução de identificar seu formato e extrair a mensagem original:
```
ZmxhZ3t0ZXN0ZX0=
```

Um dos integrantes mais avançados da equipe respondeu que se tratava de um "Hash MD5". Vamos analisar por que essa resposta é matematicamente impossível.

### 3.1 Anatomia do Base64

O Base64 foi criado para permitir que dados binários arbitrários (como imagens, arquivos executáveis ou certificados) pudessem ser transmitidos por protocolos de texto legado (como e-mail / SMTP) sem corrupção.

* **Alfabeto de 64 Símbolos:**
  * Letras maiúsculas: `A-Z` (26)
  * Letras minúsculas: `a-z` (26)
  * Dígitos: `0-9` (10)
  * Símbolos especiais: `+` e `/` (2)
  * Total = 64 caracteres ($2^6 = 6$ bits por caractere).
* **Mecânica dos 24 Bits:** O Base64 agrupa **3 bytes de 8 bits** (total de 24 bits) e os redistribui em **4 blocos de 6 bits**. Cada bloco de 6 bits mapeia um caractere da tabela.
* **O Sinal de Igual (`=`): O Padding!**
  * Se o tamanho do dado de entrada não for perfeitamente divisível por 3 bytes, sobram 1 ou 2 bytes.
  * O algoritmo preenche os bits restantes com zeros e sinaliza esse preenchimento adicionando um ou dois sinais de igual no final: `=` ou `==`.
  * **Regra Prática de CTF:** Viu caracteres alfanuméricos misturados terminando com um ou dois sinais de `=`? **É quase certeza absoluta de ser Base64!**

### 3.2 Por que NÃO poderia ser MD5?
* Um hash **MD5** possui tamanho estritamente fixo de **32 caracteres hexadecimais**. Ele só aceita dígitos de `0` a `9` e letras de `a` a `f` (geralmente todas minúsculas).
* O MD5 **nunca** possui letras como `m`, `x`, `Z`, nem símbolos como `+`, `/` e **jamais possui o caractere `=`**!

### 3.3 Decodificando no Terminal (A Resolução da Questão):

No terminal Linux, decodifica-se instantaneamente com o utilitário `base64`:

```bash
echo "ZmxhZ3t0ZXN0ZX0=" | base64 -d
# Saída exata obtida:
flag{teste}
```

Em Python:
```python
import base64

payload = "ZmxhZ3t0ZXN0ZX0="
flag = base64.b64decode(payload).decode('utf-8')
print(flag)  # flag{teste}
```

---

## 4. Hashing e Armazenamento Seguro de Senhas

No teste diagnóstico, perguntou-se qual método é adequado para armazenar senhas de usuários em bancos de dados. Mais da metade da equipe não soube responder.

### 4.1 Por que Algoritmos Rápidos (MD5, SHA-1, SHA-256) São Proibidos para Senhas?

Algoritmos da família SHA (*Secure Hash Algorithm*) foram projetados para serem **extremamente velozes**. Eles precisam calcular a integridade de arquivos de gigabytes em frações de segundo.

Essa velocidade é sua ruína quando o assunto é proteção de senhas:
* Uma única placa de vídeo gamer moderna (GPU RTX 4090) consegue calcular mais de **100 bilhões de hashes SHA-256 por segundo** com o Hashcat!
* Se o banco de dados vazar com senhas guardadas em SHA-256, um invasor quebra senhas com menos de 10 caracteres em questão de minutos através de ataques de dicionário e tabelas de pré-computação (**Rainbow Tables**).

```
VELOCIDADE DE CÁLCULO EM GPU:
SHA-256:  ~ 100.000.000.000 tentativas/segundo (INSEGURO PARA SENHAS!)
bcrypt:   ~ 10.000 a 50.000 tentativas/segundo (SEGURO E RESISTENTE!)
```

### 4.2 O que é o *Salt* e por que ele é Obrigatório?

O **Salt** (sal) é uma sequência de bytes aleatórios e únicos gerada para cada usuário no momento do cadastro. O salt é concatenado à senha antes de gerar o hash e é armazenado junto com o hash no banco.

```
SENHA DO USUÁRIO: "segredo123"
SALT ALEATÓRIO:   "k9#mP!9zL"
ENTRADA REAL:     "segredo123" + "k9#mP!9zL" ──> HASH
```

#### Os Dois Grandes Benefícios do Salt:
1. **Destruição das Rainbow Tables:** Como cada salt é único no mundo, tabelas pré-computadas de hashes tornam-se inúteis. O atacante é forçado a calcular cada tentativa individualmente.
2. **Defesa Contra Senhas Iguais:** Se dois usuários escolherem a senha `123456`, seus hashes finais serão completamente diferentes porque seus salts são distintos.

### 4.3 Os Algoritmos Modernos de Hash Lento (*Adaptive Slow Hashes*)

Para proteger senhas contra ataques massivos de GPU e ASIC, a indústria adota funções que possuem um **fator de trabalho configurável (Work Factor)**. Elas forçam o computador a despender alguns milissegundos para validar um único login:

1. **bcrypt:**  
   Baseado na cifra de bloco Blowfish. Possui um parâmetro de custo (*cost factor*) que dobra o tempo de cálculo a cada incremento numérico (ex: custo 12 executa $2^{12} = 4096$ iterações).
2. **Argon2 (Argon2id):**  
   Vencedor da *Password Hashing Competition (PHC)* internacional. É o padrão moderno recomendado pela OWASP. Ele é **memory-hard** (consome grande quantidade configurável de memória RAM), tornando inviável o ataque paralelo em GPUs, que possuem pouca memória dedicada por núcleo.
3. **PBKDF2:**  
   Padrão do NIST baseado em milhares de repetições de HMAC-SHA256.

### 4.4 Exemplo Prático de Implementação Segura em Python

Utilizando a biblioteca padrão e moderna `bcrypt`:

```python
import bcrypt

# 1. NO CADASTRO DO USUÁRIO:
senha_digitada = "MinhaSenhaSuperSecreta@2026".encode('utf-8')

# Gera um salt criptográfico aleatório com fator de custo 12
salt = bcrypt.gensalt(rounds=12)

# Gera o hash lento
senha_hash = bcrypt.hashpw(senha_digitada, salt)
print(f"Hash salvo com segurança no banco:\n{senha_hash.decode('utf-8')}")
# Saída exata: $2b$12$e7k1... (formato: $algoritmo$custo$salt+hash)

# 2. NA TENTATIVA DE LOGIN:
senha_fornecida_no_login = "MinhaSenhaSuperSecreta@2026".encode('utf-8')

# Verificação segura (constante no tempo, resistente a timing attacks)
if bcrypt.checkpw(senha_fornecida_no_login, senha_hash):
    print("[+] Autenticação autorizada com sucesso!")
else:
    print("[-] Senha incorreta!")
```

---

## 5. Resumo Rápido para CTFs da Cyberguard

Ao se deparar com strings enigmáticas em desafios de CTF:
* **Termina com `=` ou `==`? Contém letras maiúsculas, minúsculas, números e `+`, `/`?**  
  👉 Tente `base64 -d`.
* **Apenas caracteres de 0 a 9 e a até f?**  
  * 32 caracteres: **Hash MD5** ou UUID sem hífens.
  * 40 caracteres: **Hash SHA-1** ou commit do Git.
  * 64 caracteres: **Hash SHA-256**.
  * Texto legível com caracteres aos pares (ex: `666c6167`): **Hexadecimal (ASCII Hex)** (`xxd -r -p` para converter de volta a texto).
* **Cifras de Substituição / Rotação (ROT13 / César):** O formato da flag `flag{...}` aparece mantendo as pontuações e tamanhos (ex: `synt{...}`). Use o utilitário online **CyberChef** (*The Cyber Swiss Army Knife*) para testar rotações instantâneas.
