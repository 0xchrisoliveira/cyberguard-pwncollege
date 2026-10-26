Vamos começar imprimindo o conteúdo de variáveis na tela.
O programa `/challenge/run` não entregará — e não tem como entregar — a flag para você, mas tudo bem: a flag já foi gravada na variável chamada "FLAG"!
Basta fazer sua shell imprimi-la!

Existem várias formas de fazer isso, mas começaremos com o `echo`.
Esse comando simplesmente imprime dados no terminal.
Por exemplo:

```console
hacker@dojo:~$ echo Hello Hackers!
Hello Hackers!
```

Você também pode imprimir variáveis com o `echo`, bastando prefixar o nome da variável com o caractere `$`.
Por exemplo, existe uma variável chamada `PWD` que sempre armazena o diretório de trabalho atual da shell.
Você pode visualizá-la assim:

```console
hacker@dojo:~$ echo $PWD
/home/hacker
```

Agora é a sua vez.
Faça sua shell exibir a variável `FLAG` e resolva este desafio!
