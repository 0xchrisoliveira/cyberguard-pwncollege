Um dos comandos mais fundamentais do Linux é o `cat`.
O `cat` é usado com maior frequência para ler o conteúdo de arquivos, desta forma:

```console
hacker@dojo:~$ cat /path/to/file
Hello Hackers!
```

O `cat` con**cat**ena (daí a origem do seu nome) múltiplos arquivos se você fornecer múltiplos argumentos.
Por exemplo:

```console
hacker@dojo:~$ cat myfile
This is my file!
hacker@dojo:~$ cat yourfile
This is your file!
hacker@dojo:~$ cat myfile yourfile
This is my file!
This is your file!
hacker@dojo:~$ cat myfile yourfile myfile
This is my file!
This is your file!
This is my file!
```

Por fim, se você não passar nenhum argumento, o `cat` lerá a entrada digitada no próprio terminal e a ecoará de volta na saída.
Exploraremos esse comportamento em desafios futuros...

Neste desafio, a flag foi copiada para o arquivo `flag` no seu diretório home (onde a sua shell é iniciada).
Vá até lá e leia-o com o comando `cat`!
