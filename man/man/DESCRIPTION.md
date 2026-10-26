Este nível apresenta o comando `man`.
`man` é uma abreviação de `manual`, e exibe (quando disponível) o manual de instruções do comando passado como argumento.
Por exemplo, imagine que queiramos aprender sobre o comando `yes` (_sim_, este é um comando real do Linux):

```console
hacker@dojo:~$ man yes
```

Isso exibirá a página de manual (*man page*) do `yes`, parecida com isto:

```text
YES(1)                           User Commands                          YES(1)

NAME
       yes - output a string repeatedly until killed

SYNOPSIS
       yes [STRING]...
       yes OPTION

DESCRIPTION
       Repeatedly output a line with all specified STRING(s), or 'y'.

       --help display this help and exit

       --version
              output version information and exit

AUTHOR
       Written by David MacKenzie.

REPORTING BUGS
       GNU coreutils online help: <https://www.gnu.org/software/coreutils/>
       Report any translation bugs to <https://translationproject.org/team/>

COPYRIGHT
       Copyright  ©  2020  Free Software Foundation, Inc.  License GPLv3+: GNU
       GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
       This is free software: you are free  to  change  and  redistribute  it.
       There is NO WARRANTY, to the extent permitted by law.

SEE ALSO
       Full documentation <https://www.gnu.org/software/coreutils/yes>
       or available locally via: info '(coreutils) yes invocation'

GNU coreutils 8.32               February 2022                          YES(1)
```

As seções mais importantes de uma manpage são:

```text
NOME(1)                           CATEGORIA                         NOME(1)

NAME (NOME)
	Informa o nome e uma breve descrição do comando ou conceito
	abordado na página.

SYNOPSIS (SINOPSE)
	Apresenta uma sinopse resumida da sintaxe de uso. Essas sinopses
	seguem um formato padrão. Tipicamente, cada linha é uma invocação
	válida do comando, podendo ser interpretada como:

	COMANDO [ARGUMENTO_OPCIONAL] ARGUMENTO_OBRIGATORIO_UNICO
	COMANDO [ARGUMENTO_OPCIONAL] MULTIPLOS_ARGUMENTOS...

DESCRIPTION (DESCRIÇÃO)
	Detalhamento completo do comando ou conceito, com descrições
	minuciosas de cada uma das opções disponíveis.

SEE ALSO (VEJA TAMBÉM)
	Outras páginas de manual ou recursos online que podem ser úteis.

COLEÇÃO                           DATA                              NOME(1)
```

Você pode navegar pela manpage usando as setas do teclado (↑ / ↓) e as teclas PgUp / PgDn.
Quando terminar de ler o manual, pressione `q` para sair e voltar ao terminal.

As páginas man ficam armazenadas em uma base de dados centralizada do sistema.
Por curiosidade, essa base reside no diretório `/usr/share/man`, mas você nunca precisará acessá-la diretamente: basta consultá-la usando o comando `man`.
Os argumentos passados ao `man` não são caminhos de arquivos, mas apenas os nomes das entradas (por exemplo, execute `man yes` para consultar o manual do `yes`, em vez de `man /usr/bin/yes`, que é o caminho real do executável mas faria o `man` falhar ou exibir caracteres ilegíveis).

O desafio deste nível possui uma opção secreta que, quando informada, fará o programa imprimir a flag.
Você deve descobrir qual é essa opção lendo a página de manual do `challenge`!
