Anteriormente, seu caminho relativo foi "direto": ele especificou sem rodeios o diretório no qual navegar a partir do diretório atual.
Neste nível, vamos explorar caminhos relativos mais explícitos.

Na maioria dos sistemas operacionais, incluindo o Linux, todo diretório possui duas entradas implícitas que você pode referenciar em caminhos: `.` e `..`.
A primeira, `.`, aponta diretamente para o próprio diretório atual, de modo que os seguintes caminhos absolutos são todos idênticos entre si:

- `/challenge`
- `/challenge/.`
- `/challenge/./././././././././`
- `/./././challenge/././`

Os seguintes caminhos relativos também são todos idênticos entre si:

- `challenge`
- `./challenge`
- `./././challenge`
- `challenge/.`

Naturalmente, se o seu diretório atual de trabalho for `/`, os caminhos relativos acima equivalem exatamente aos caminhos absolutos correspondentes.

Este desafio fará você utilizar o `.` nos seus caminhos relativos.
Seu diretório atual de trabalho deve ser `/`, e o caminho relativo que você usar para invocar o desafio deve começar obrigatoriamente com `.`.
Prepare-se!
