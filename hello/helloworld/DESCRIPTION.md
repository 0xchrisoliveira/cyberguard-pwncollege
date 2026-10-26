Vamos tentar algo mais elaborado: um comando com _argumentos_, que é como chamamos os dados adicionais passados para um comando.
Quando você digita uma linha de texto e pressiona Enter, a shell processa a sua entrada dividindo-a entre o comando e seus _argumentos_.
A primeira palavra é o comando, e as palavras seguintes são os argumentos.
Observe:

```console
hacker@dojo:~$ echo Hello
Hello
hacker@dojo:~$
```

Neste caso, o comando foi `echo`, e o argumento foi `Hello`.
O `echo` é um comando simples que "ecoa" todos os seus argumentos de volta no terminal, como você vê na sessão acima.

Vejamos o `echo` com múltiplos argumentos:

```console
hacker@dojo:~$ echo Hello Hackers!
Hello Hackers!
hacker@dojo:~$
```

Neste caso, o comando foi `echo`, e `Hello` e `Hackers!` foram os dois argumentos passados ao `echo`.
Simples assim!

Neste desafio, para obter a flag, você deve executar o comando `hello` (NÃO o comando `echo`) com um único argumento: `hackers`.
Tente agora!
