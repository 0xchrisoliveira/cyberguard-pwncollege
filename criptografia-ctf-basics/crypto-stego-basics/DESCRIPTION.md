# Esteganografia e Análise de Metadados em Imagens

### 📖 Introdução Teórica
A palavra **Esteganografia** tem origem grega (*steganos* = oculto / protegido e *graphein* = escrita) e designa a técnica de ocultar a existência de uma mensagem dentro de um arquivo portador (*carrier* ou hospedeiro), de tal modo que terceiros não suspeitem de que há dados secretos em trânsito.

**Diferença Crucial entre Esteganografia e Criptografia:**
* **Criptografia:** Transforma a mensagem legível em um texto incompreensível (*ciphertext*). Qualquer observador sabe que uma mensagem cifrada está passando, mas não consegue ler sem a chave.
* **Esteganografia:** Dissimula a própria presença da mensagem. O observador vê apenas uma foto inocente, uma música MP3 ou um documento binário legítimo. Frequentemente, equipes ofensivas combinam ambas as técnicas: cifram o payload com AES e depois o esteganografam em uma imagem.

**Técnicas Comuns de Ocultação Digital:**
1. **Dados Injetados após o Marcador de Fim de Arquivo (EOF - End of File):**
   Formatos de imagem possuem delimitadores estritos de terminação:
   - Arquivos PNG terminam com o chunk `IEND` (`IEND®B``).
   - Arquivos JPEG terminam com o marcador hexadecimal `ÿÙ`.
   Quaisquer bytes anexados após essa assinatura de encerramento são ignorados pelos visualizadores gráficos usuais, mas continuam fisicamente armazenados no disco, permitindo transporte de arquivos compactados e chaves.
2. **Metadados e Tags EXIF:**
   Arquivos de mídia contêm seções para informações complementares (modelo da câmera, timestamps, coordenadas de GPS e campos de comentário). Invasores podem embutir textos e comandos codificados diretamente no campo `Comment` ou `Artist`.
3. **LSB (Least Significant Bit - Bit Menos Significativo):**
   Substitui o último bit de cada byte de componente de cor (Red, Green, Blue) de uma imagem pelo bit do payload secreto. A alteração de 1 bit no valor do pixel (ex: de 254 para 255) causa uma variação de cor imperceptível ao olho humano, mas possibilita esconder mensagens inteiras nos planos de bits.
4. **Arquivos Poliglotas e Concatenação:**
   Técnica de embutir um arquivo ZIP dentro de uma imagem PNG (`cat imagem.png segredo.zip > portador.png`). O arquivo abre perfeitamente como imagem e simultaneamente pode ser descompactado com utilitários de compressão.

### 💻 Exemplos Práticos e Sintaxe
Ferramentas essenciais no terminal Linux para triagem esteganográfica:

1. **Inspecionar Magic Bytes e Tipo Real de Arquivo (`file`):**
   ```bash
   file /tmp/suspicious.bin
   ```
   *Permite saber se um arquivo disfarçado com extensão `.jpg` é na verdade um executável ELF ou um arquivo compactado.*

2. **Extrair Strings Legíveis com Comprimento Mínimo (`strings`):**
   ```bash
   strings -n 6 /tmp/suspicious.bin | grep -i flag
   ```
   *A flag `-n 6` filtra apenas sequências imprimíveis com 6 ou mais caracteres consecutivos.*

3. **Inspecionar Assinaturas e Carregadores com `binwalk`:**
   ```bash
   # Escanear por assinaturas de arquivos embutidos (ZIP, GZ, ELF, etc.):
   binwalk /caminho/arquivo.png

   # Extrair automaticamente qualquer arquivo compactado encontrado dentro da imagem:
   binwalk -e /caminho/arquivo.png
   ```

4. **Examinar Metadados Profundos com `exiftool`:**
   ```bash
   exiftool /tmp/suspicious.bin
   ```
   *Exibe tags EXIF, dados de geolocalização e comentários embutidos.*

5. **Análise de Planos LSB em Imagens PNG com `zsteg`:**
   ```bash
   zsteg -a imagem.png
   ```
   *Testa automaticamente todas as combinações de canais de cor e ordens de bits LSB.*

6. **Inspeção Hexadecimal de Cabeçalhos com `xxd` ou `hexdump`:**
   ```bash
   xxd -l 64 /tmp/suspicious.bin
   ```

### 🎯 Seu Objetivo

Investigar um arquivo de mídia digital (imagem) que oculta dados confidenciais em sua estrutura interna. Seu objetivo é aplicar técnicas básicas de esteganografia e análise de artefatos (como inspeção de sequências de strings imprimíveis ou extração de metadados e blocos anexados) para revelar a mensagem secreta oculta na imagem e obter a flag.
