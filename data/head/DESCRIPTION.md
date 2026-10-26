Em sua jornada pelo Linux, você passará por inúmeras situações em que precisará capturar apenas as primeiras linhas da saída de programas bastante verbosos.
Para isso, você usará o comando `head`!
O `head` é utilizado para exibir as linhas iniciais de sua entrada:

```console
hacker@dojo:~$ cat /something/very/long | head
this
is
just
the
first
ten
lines
of
the
file
hacker@dojo:~$
```

Por padrão, ele exibe as 10 primeiras linhas, mas você pode definir a quantidade desejada através da opção `-n`:

```console
hacker@dojo:~$ cat /something/very/long | head -n 2
this
is
hacker@dojo:~$
```

Neste desafio, `/challenge/pwn` gera uma grande quantidade de dados, e você precisará canalizar essa saída com pipe para o `head`, pegando apenas as 7 primeiras linhas, e em seguida canalizar essas linhas adiante para `/challenge/college`, que entregará a flag caso o procedimento esteja correto!
Sua solução consistirá em um pipeline composto conectando três comandos por meio de *dois* pipes (`|`).
Boa sorte!
