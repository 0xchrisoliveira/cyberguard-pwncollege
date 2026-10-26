# Esteganografia e Análise de Metadados em Arquivos (Crypto Stego Basics)

## Briefing da Missão
* **Codinome:** Operação Invisible Payload
* **Classificação:** Confidencial / Treinamento de Esteganografia & Forense em Mídia
* **Alvo:** Arquivo Binário Transportador (`/tmp/suspicious.bin`)
* **Objetivo:** Inspecionar a estrutura de bytes e metadados de um arquivo suspeito utilizando comandos de análise no terminal Linux (`file`, `strings`, `binwalk`, `exiftool`) para localizar e extrair a chave secreta esteganografada.

Em cenários de CTF e investigações de espionagem cibernética, agentes adversários ocultam dados sensíveis dentro de mídias que aparentam ser arquivos comuns de imagem ou documento. Desenvolver reflexos de linha de comando para rápida triagem de arquivos é indispensável para operadores de segurança.

---

## Teoria Aprofundada

### 1. Esteganografia vs Criptografia
Enquanto a criptografia garante que ninguém além do detentor da chave possa compreender o conteúdo de uma mensagem, a **esteganografia esconde a própria existência do canal de comunicação**:

```
[ CRIPTOGRAFIA ]
Mensagem clara: "Ataque as 03h" ──(AES-256)──> Ciphertext: "7a#9!x&L"
Inimigo intercepta: "Há uma mensagem cifrada aqui, tentem quebrar!"

[ ESTEGANOGRAFIA ]
Mensagem clara: "Ataque as 03h" ──(Ocultação em Bytes)──> Foto de um Gato (PNG)
Inimigo intercepta: "Apenas uma foto inofensiva de animal de estimação."
```

### 2. Mecanismos Populares de Esteganografia Digital

```
┌────────────────────────────────────────────────────────────────────────┐
│ 1. DADOS APÓS O FIM DE ARQUIVO (EOF - End Of File)                     │
│ Imagens possuem delimitadores formais de terminação:                   │
│ - JPEG: \xFF\xD9                                                       │
│ - PNG:  IEND \x49\x45\x4E\x44\xAE\x42\x60\x82                         │
│ Quaisquer bytes concatenados DEPOIS dessa marcação são ignorados       │
│ pelo visualizador gráfico, mas continuam fisicamente no disco!         │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
┌──────────────────────────────────┴─────────────────────────────────────┐
│ 2. METADADOS E TAGS EXIF                                               │
│ Campos de comentário (Comment), Autor, Criador ou coordenadas GPS      │
│ podem ser usados para transportar strings e chaves codificadas.        │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
┌──────────────────────────────────┴─────────────────────────────────────┐
│ 3. LSB (LEAST SIGNIFICANT BIT)                                         │
│ Substitui o último bit de cada canal de cor (Red, Green, Blue).        │
│ A mudança na cor do pixel é matematicamente indetectável a olho nu.    │
└────────────────────────────────────────────────────────────────────────┘
```

### 3. Tabela Comparativa de Utilitários de Linha de Comando

| Ferramenta | Finalidade Principal | Exemplo de Comando | O que Procura |
| :--- | :--- | :--- | :--- |
| **`file`** | Detecção de Magic Bytes | `file /tmp/suspicious.bin` | Verifica se a extensão condiz com o tipo real |
| **`strings`** | Extração de texto legível | `strings -n 6 arquivo \| grep -i flag` | Segredos em texto plano, comentários embutidos |
| **`binwalk`** | Busca de assinaturas de arquivos | `binwalk -e arquivo.png` | Arquivos compactados (ZIP, GZ) concatenados |
| **`exiftool`**| Leitura profunda de metadados | `exiftool foto.jpg` | Câmera, comentários, geolocalização, timestamps |
| **`zsteg`** | Análise de canais LSB em PNG | `zsteg -a imagem.png` | Payloads ocultos nos planos de bits de cores |

---

## Passo a Passo de Resolução

1. Execute o desafio para inicializar o ambiente e gerar o artefato transportador:
   ```bash
   /challenge/crypto-stego-basics
   ```
2. Caso queira consultar a aula técnica:
   ```bash
   /challenge/crypto-stego-basics --help
   ```
3. O desafio gerou o arquivo de análise em `/tmp/suspicious.bin`. Utilize o utilitário `strings` combinado com `grep` para filtrar strings que contenham o identificador `FLAG_HIDDEN_STEGO`:
   ```bash
   strings /tmp/suspicious.bin | grep FLAG_HIDDEN_STEGO
   ```
   A saída exibirá a linha contendo o segredo:
   ```text
   ### METADATA_PAYLOAD: FLAG_HIDDEN_STEGO_cyberguard2026 ###
   ```
4. No prompt do desafio, informe a chave extraída:
   ```text
   cyberguard2026
   ```
5. Você também pode submeter o segredo diretamente pela linha de comando:
   ```bash
   /challenge/crypto-stego-basics cyberguard2026
   ```
6. O sistema confirmará a integridade do artefato e exibirá a flag do desafio.

---

## Dicas e Comandos Práticos

* **Inspecionando os Magic Bytes com `hexdump` / `xxd`:**
  ```bash
  # Exibir os primeiros 16 bytes do arquivo em hexadecimal e ASCII:
  xxd -l 32 /tmp/suspicious.bin
  ```
  Note a assinatura PNG nos primeiros bytes: `\x89 P N G \r \n \x1a \n`.
* **Extraindo arquivos ocultos com `binwalk`:**
  Se um invasor concatenar um arquivo `.zip` dentro de uma imagem `.jpg` (`cat foto.jpg arquivo.zip > payload.jpg`), o visualizador abre a foto normalmente, mas o comando `binwalk -e payload.jpg` detecta a assinatura PKZIP e descompacta o arquivo oculto na pasta `_payload.jpg.extracted/`.
* **Steghide para arquivos com senha:**
  ```bash
  # Extrair dados de fotos protegidas com passphrase via steghide:
  steghide extract -sf imagem.jpg -p "senha123"
  ```

---

## Aplicação Prática no Mundo Real
Grupos de Ameaça Persistente Avançada (APTs, como o Lazarus Group e Turla) utilizam canais encobertos (*covert channels*) com esteganografia para duas finalidades principais:
1. **Bypass de Antivírus e EDR:** Fazer o download de shellcodes em partes dentro de logotipos de empresas no formato PNG/BMP hospedados em servidores públicos legítimos (como o GitHub ou Imgur).
2. **Exfiltração de Segredos:** Dados corporativos confidenciais são embutidos em fotografias de eventos ou produtos e enviados para fora da organização via e-mail ou redes sociais, contornando sistemas corporativos de DLP (*Data Loss Prevention*).
