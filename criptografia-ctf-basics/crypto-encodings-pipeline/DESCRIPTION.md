# Decodificando Encodings em Cascata

### 📖 Introdução Teórica
Uma das defasagens conceituais mais comuns na segurança da informação é a confusão entre **Codificação (Encoding)**, **Resumo Criptográfico (Hashing)** e **Criptografia Real (Encryption)**. Dominar essas distinções é indispensável:

| Conceito | Finalidade Primária | Reversibilidade | Depende de Chave Secreta? | Exemplos Canônicos |
| :--- | :--- | :---: | :---: | :--- |
| **Encoding (Codificação)** | Compatibilidade e transmissão de dados binários em canais que aceitam apenas texto | **100% Reversível** | ❌ **NÃO** | Base64, Hexadecimal, ASCII, URL-Encoding, ROT13 |
| **Hashing (Resumo Unidirecional)** | Garantir integridade e gerar impressões digitais matemáticas de tamanho fixo | **NÃO Reversível** (One-Way) | ❌ **NÃO** | MD5, SHA-1, SHA-256 |
| **Encryption (Criptografia Real)** | Proteger confidencialidade contra acesso não autorizado | **Reversível** (com a chave) |  **SIM** | AES-GCM, ChaCha20, RSA, ECC |

**Anatomia Matemática do Base64 e a Assinatura do Padding (`=`):**
O Base64 foi desenvolvido para empacotar dados binários de 8 bits em um subconjunto de 64 caracteres ASCII seguros para transmissão em protocolos de texto (como e-mail MIME ou JSON).
* O alfabeto é formado por 64 símbolos: `A-Z` (26 letras maiúsculas), `a-z` (26 letras minúsculas), `0-9` (10 dígitos), `+` e `/`. Cada símbolo codificado representa exatamente 6 bits ($2^6 = 64$).
* O algoritmo agrupa blocos de 3 bytes da entrada ($3 	imes 8 = 24$ bits) e os divide em 4 blocos de 6 bits, mapeando cada um no caractere correspondente.
* **Padding (`=` e `==`):** Quando a entrada original não é perfeitamente divisível por 3 bytes:
  - Sobra 1 byte (8 bits): precisa de 2 caracteres Base64 (12 bits) e é completado com dois sinais de igual (`==`) como preenchimento.
  - Sobram 2 bytes (16 bits): precisa de 3 caracteres Base64 (18 bits) e é completado com um sinal de igual (`=`).
  - Múltiplo exato de 3 bytes: nenhum caractere de preenchimento é gerado.

**Por que a string `ZmxhZ3t0ZXN0ZX0=` JAMAIS poderia ser um Hash MD5?**
1. **Conjunto de Caracteres:** Hashes MD5 utilizam exclusivamente caracteres hexadecimais (`0-9`, `a-f`). A string acima contém caracteres como `Z`, `m`, `x`, `t` que não existem na base hexadecimal.
2. **Sinal de Padding (`=`):** O caractere `=` NUNCA existe em um hash MD5, SHA-1 ou SHA-256.
3. **Comprimento:** Um hash MD5 possui sempre exatamente 32 caracteres hexadecimais (128 bits).

### 💻 Exemplos Práticos e Sintaxe
Utilitários essenciais no terminal Linux para manipulação de encodings:

1. **Decodificar Base64 com o comando nativo `base64` (`-d` ou `--decode`):**
   ```bash
   echo -n "ZmxhZ3t0ZXN0ZX0=" | base64 -d
   ```
   *Saída esperada no terminal:*
   ```text
   flag{teste}
   ```

2. **Codificar strings em Base64:**
   ```bash
   echo -n "flag{teste}" | base64
   ```
   *Saída esperada:*
   ```text
   ZmxhZ3t0ZXN0ZX0=
   ```

3. **Decodificar Hexadecimal para ASCII com `xxd` (`-r -p`):**
   ```bash
   echo -n "666c61677b74657374657d" | xxd -r -p
   ```
   *Saída esperada:*
   ```text
   flag{teste}
   ```

4. **Decodificar ROT13 com `tr` no terminal:**
   ```bash
   echo "synt{grfgr}" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
   ```

5. **Decodificação Programática em Python:**
   ```python
   import base64

   payload = "ZmxhZ3t0ZXN0ZX0="
   resultado = base64.b64decode(payload).decode("utf-8")
   print("Texto recuperado:", resultado)
   ```

### 🎯 O que fazer para resolver o desafio (Passo a Passo)
1. **Localizar e Iniciar o Binário:**
   No terminal do container, execute o executável do desafio:
   ```bash
   /challenge/crypto-encodings-pipeline
   ```
   O binário apresentará a string interceptada: `ZmxhZ3t0ZXN0ZX0=`.
2. **Decodificar o Payload no Terminal:**
   Em uma linha de comando do Linux, decodifique a string com o utilitário nativo:
   ```bash
   echo -n "ZmxhZ3t0ZXN0ZX0=" | base64 -d
   ```
   O comando exibirá o texto original em claro:
   ```text
   flag{teste}
   ```
3. **Enviar a Resposta ao Desafio:**
   No prompt do binário (`>> `), digite o texto em claro recuperado:
   ```text
   flag{teste}
   ```
   *Alternativa rápida:* Você pode executar o binário passando a resposta diretamente:
   ```bash
   /challenge/crypto-encodings-pipeline "flag{teste}"
   ```
4. **Capturar e Submeter a Flag:**
   O programa validará a decodificação da string Base64 e imprimirá a flag no formato:
   ```text
   pwn.college{...}
   ```
   Copie a flag e registre a pontuação na plataforma.
