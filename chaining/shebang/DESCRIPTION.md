Você está avançando rápido na sua jornada com shell scripting!
No entanto, até agora, os seus shell scripts *só podem ser iniciados de dentro da shell*.
As coisas funcionaram perfeitamente no nível anterior porque você estava invocando o script a partir da shell interativa `bash`, mas isso falharia se o seu script fosse invocado diretamente por um processo do sistema operacional, por exemplo, um programa escrito em Python, C ou qualquer outra linguagem.

Quando um programa é executado no Linux, o kernel inspeciona os primeiros bytes do arquivo para determinar como ele deve ser executado.
O kernel NÃO se baseia na extensão do arquivo (por isso você não é *obrigado* a nomear seus scripts com `.sh`, scripts Python com `.py`, e assim por diante).
Em vez disso, o Linux analisa os primeiros bytes do conteúdo do arquivo.

Existem vários tipos de executáveis, mas se o arquivo começar com a sequência de caracteres mágicos `#!` (conhecida como "[shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))"), o Linux reconhece o arquivo como um *programa interpretado* e trata o restante da linha como o caminho absoluto para o *interpretador*.
Em seguida, o kernel invoca esse interpretador passando o caminho do script como argumento.

Considere este shell script:

```bash
#!/bin/bash

echo "Hello Hackers!"
```

Ele pode ser executado assim:

```console
hacker@dojo:~$ chmod a+x script.sh
hacker@dojo:~$ ./script.sh
Hello Hackers!
hacker@dojo:~$
```

Quando `./script.sh` foi executado, o kernel do Linux abriu o arquivo, leu a primeira linha, extraiu `/bin/bash` como o interpretador e executou `/bin/bash ./script.sh` para iniciar o script!

Atenção: a linha da shebang deve ser OBRIGATORIAMENTE a PRIMEIRÍSSIMA linha do arquivo — sem linhas em branco e sem espaços antes dela!

Para este desafio, crie um script em `/home/hacker/solve.sh` que contenha uma shebang adequada e imprima o texto "hack the planet".
Lembre-se de torná-lo executável e, em seguida, execute `/challenge/run` para validar se o seu script funciona corretamente!

----
**CURIOSIDADE:**
Shebangs muito comuns que você encontrará:
- `#!/bin/bash` para scripts em Bash
- `#!/usr/bin/python3` para scripts em Python 3
- `#!/bin/sh` para scripts na shell padrão POSIX — um predecessor mais básico do `bash` com menos recursos nativos, mas com máxima compatibilidade entre sistemas Unix/Linux!
