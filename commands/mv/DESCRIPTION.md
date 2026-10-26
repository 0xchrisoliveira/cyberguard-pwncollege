Você também pode _mover_ e renomear arquivos usando o comando `mv` (*move*).
O uso é muito simples:

```console
hacker@dojo:~$ ls
my-file
hacker@dojo:~$ cat my-file
PWN!
hacker@dojo:~$ mv my-file your-file
hacker@dojo:~$ ls
your-file
hacker@dojo:~$ cat your-file
PWN!
hacker@dojo:~$
```

Este desafio quer que você mova o arquivo `/flag` para `/tmp/hack-the-planet` (faça isso)!
Atenção: você _deve_ obrigatoriamente usar o comando `mv` em vez de outros métodos (como renomear o arquivo graficamente pelo VSCode).
Quando terminar, execute `/challenge/check`, que verificará a operação e entregará a flag a você.
