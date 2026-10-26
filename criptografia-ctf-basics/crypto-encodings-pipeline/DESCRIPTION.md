# Decodificando Encodings em Cascata (Crypto Encodings Pipeline)

## Briefing da Missão
* **Codinome:** Operação Turing Cascade
* **Classificação:** Confidencial / Treinamento de Criptoanálise & CTF
* **Alvo:** Fluxo de Comunicações e Cargas Codificadas da Cyberguard
* **Objetivo:** Identificar a assinatura visual e matemática da codificação Base64, diferenciar formatos de codificação de funções de hash e decodificar a mensagem original `ZmxhZ3t0ZXN0ZX0=`.

Na avaliação diagnóstica 2026.2 da Cyberguard, a **questão 20** expôs uma confusão alarmante: **75% dos membros da equipe** não soube identificar a codificação Base64 ou errou a decodificação da string, com integrantes chegando a confundi-la com um hash MD5. Este desafio fundamenta a mecânica de encodings e consolida o uso de utilitários de linha de comando no Linux.

---

## Teoria Aprofundada

### 1. A Tríade da Informação: Encoding vs Hashing vs Encryption
Na segurança ofensiva e defensiva, é inaceitável usar esses três conceitos de forma intercambiável:

```
+-------------------+------------------------------------+--------------------------+
| CONCEITO          | OBJETIVO PRINCIPAL                 | REVERSIBILIDADE          |
+-------------------+------------------------------------+--------------------------+
| 1. ENCODING       | Compatibilidade de transporte      | 100% Reversível (SEM     |
| (Codificação)     | de dados binários em canais texto  | chave alguma necessária) |
+-------------------+------------------------------------+--------------------------+
| 2. HASHING        | Integridade e impressão digital    | UNIDIRECIONAL (One-Way)  |
| (Resumo Cripto)   | de tamanho fixo de um arquivo      | Impossível reverter      |
+-------------------+------------------------------------+--------------------------+
| 3. ENCRYPTION     | Confidencialidade e sigilo         | Reversível APENAS por    |
| (Cifragem Real)   | contra acessos não autorizados     | quem detém a CHAVE certa |
+-------------------+------------------------------------+--------------------------+
```

### 2. Anatomia Matemática do Base64
O Base64 foi concebido para empacotar dados binários de 8 bits em um subconjunto restrito de 64 caracteres ASCII seguros para transmissão em canais legados (como e-mail MIME).

* **Tamanho do Alfabeto:** $2^6 = 64$ símbolos (`A-Z`, `a-z`, `0-9`, `+`, `/`).
* **Mapeamento de 24 Bits:** O algoritmo toma 3 bytes de entrada (24 bits) e os particiona em 4 blocos de 6 bits:

```
Entrada:   [ Byte 1: 8 bits ] [ Byte 2: 8 bits ] [ Byte 3: 8 bits ]  (Total: 24 bits)
               │                   │                   │
               └─────────┬─────────┴─────────┬─────────┘
                         ▼                   ▼
Saída:     [Char 1: 6b] [Char 2: 6b] [Char 3: 6b] [Char 4: 6b]        (Total: 24 bits)
```

### 3. A Assinatura do Padding (`=` e `==`)
Quando a entrada original não é perfeitamente divisível por 3 bytes:
* **Sobra 1 byte (8 bits):** Precisa de 2 caracteres Base64 (12 bits) e é completado com dois sinais de preenchimento (`==`).
* **Sobram 2 bytes (16 bits):** Precisa de 3 caracteres Base64 (18 bits) e é completado com um sinal de preenchimento (`=`).
* **Múltiplo de 3 bytes:** Nenhum caractere de padding é gerado.

> **Regra de Ouro em CTFs:** Viu uma cadeia alfanumérica contendo letras maiúsculas, minúsculas e terminando com `=` ou `==`? **Trata-se de Base64.**

### 4. Por que `ZmxhZ3t0ZXN0ZX0=` JAMAIS poderia ser um Hash MD5?

| Característica | Base64 | Hash MD5 |
| :--- | :--- | :--- |
| **Conjunto de Caracteres** | `A-Z`, `a-z`, `0-9`, `+`, `/`, `=` | Estritamente Hexadecimal: `0-9`, `a-f` |
| **Letras Permitidas** | Qualquer letra maiúscula ou minúscula | Somente letras de `a` até `f` (geralmente minúsculas) |
| **Sinal de Igual (`=`)** | Usado como preenchimento de padding | **NUNCA existe em hashes MD5** |
| **Comprimento** | Variável (múltiplo de 4 caracteres) | Fixo: exatamente 32 caracteres (128 bits) |

---

## Passo a Passo de Resolução

1. Execute o desafio para inspecionar o payload:
   ```bash
   /challenge/crypto-encodings-pipeline
   ```
2. Caso precise consultar a aula teórica:
   ```bash
   /challenge/crypto-encodings-pipeline --help
   ```
3. No terminal do Linux, utilize o utilitário nativo `base64` com a flag de decodificação (`-d` ou `--decode`):
   ```bash
   echo -n "ZmxhZ3t0ZXN0ZX0=" | base64 -d
   ```
   A saída no terminal será:
   ```text
   flag{teste}
   ```
4. No prompt do desafio, insira a resposta decodificada:
   ```text
   flag{teste}
   ```
5. Ou execute o binário diretamente com a resposta via argumento:
   ```bash
   /challenge/crypto-encodings-pipeline "flag{teste}"
   ```
6. O sistema validará a decodificação e exibirá a flag do desafio.

---

## Dicas e Comandos Práticos

* **Decodificação em Python (One-Liner):**
  ```bash
  python3 -c "import base64; print(base64.b64decode('ZmxhZ3t0ZXN0ZX0=').decode())"
  ```
* **Decodificando Pipelines em Cadeia (Hexadecimal + Base64):**
  Em desafios mais complexos, um atacante pode aninhar múltiplos encodings:
  ```bash
  # Hexadecimal para binário e depois Base64:
  echo "5a6d78685a3374305a584e305a58303d" | xxd -r -p | base64 -d
  ```
* **CyberChef:**
  A ferramenta web *CyberChef* (The Cyber Swiss Army Knife) é a referência mundial para desembaraçar múltiplos níveis de encriptação, descompressão (gzip) e encodings sucessivos em CTFs.

---

## Aplicação Prática no Mundo Real
O Base64 é amplamente utilizado na internet:
1. **Cabeçalhos de Autenticação HTTP:** A autenticação HTTP Basic envia credenciais no formato `Authorization: Basic dXNlcjpzZW5oYQ==`. Como Base64 não é criptografia, qualquer interceptador de rede recupera usuário e senha instantaneamente caso não haja TLS (HTTPS).
2. **Payloads Ofensivos de PowerShell:** Invasores utilizam `powershell -EncodedCommand <Base64>` para evitar que comandos com aspas e caracteres especiais sejam corrompidos pela linha de comando.
3. **Anexos de E-mail (MIME):** Arquivos PDF e imagens são convertidos em Base64 para trafegarem de forma confiável no protocolo SMTP.
