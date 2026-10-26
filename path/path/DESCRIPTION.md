Acontece que a resposta para "Como a shell encontra o `ls`?" é bastante simples.
Existe uma variável especial de shell, chamada `PATH`, que armazena uma lista de diretórios nos quais a shell procurará pelos programas correspondentes aos comandos digitados.
Se você esvaziar essa variável, as coisas começam a falhar:

```console
hacker@dojo:~$ ls
Desktop    Downloads  Pictures  Templates
Documents  Music      Public    Videos
hacker@dojo:~$ PATH=""
hacker@dojo:~$ ls
bash: ls: No such file or directory
hacker@dojo:~$
```

Sem a variável PATH, o bash não consegue encontrar o comando `ls`.

Neste nível, você irá interferir no funcionamento do programa `/challenge/run`.
Esse programa tentará **EXCLUIR** o arquivo da flag usando o comando `rm`.
No entanto, se ele não conseguir encontrar o comando `rm`, a flag não será excluída e o desafio a entregará para você!
Portanto, você deve fazer com que o `/challenge/run` também não consiga encontrar o comando `rm`!

Lembre-se: se você não conseguir e a flag for apagada, será necessário reiniciar o desafio para tentar novamente!
