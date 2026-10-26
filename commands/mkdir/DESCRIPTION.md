Nós já sabemos criar arquivos.
Mas e quanto a diretórios (pastas)?
Você cria diretórios (**m**a**k**e **dir**ectories) usando o comando `mkdir`.
Depois, você pode colocar arquivos dentro deles!

Veja:

```console
hacker@dojo:~$ cd /tmp
hacker@dojo:/tmp$ ls
hacker@dojo:/tmp$ ls
hacker@dojo:/tmp$ mkdir my_directory
hacker@dojo:/tmp$ ls
my_directory
hacker@dojo:/tmp$ cd my_directory
hacker@dojo:/tmp/my_directory$ touch my_file
hacker@dojo:/tmp/my_directory$ ls
my_file
hacker@dojo:/tmp/my_directory$ ls /tmp/my_directory/my_file
/tmp/my_directory/my_file
hacker@dojo:/tmp/my_directory$
```

Agora, avance e crie um diretório chamado `/tmp/pwn` e crie um arquivo chamado `college` dentro dele!
Em seguida, execute `/challenge/run`, que verificará a sua solução e entregará a flag a você!
