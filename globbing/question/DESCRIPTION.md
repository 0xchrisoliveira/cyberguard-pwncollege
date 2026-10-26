A seguir, vamos aprender sobre o `?`.
Quando a shell encontra um caractere `?` em qualquer argumento, ela o trata como um coringa de **um único caractere**.
Ele funciona como o `*`, mas corresponde a exatamente *um* caractere.
Por exemplo:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_cc
hacker@dojo:~$ ls
file_a	file_b	file_cc
hacker@dojo:~$ echo Look: file_?
Look: file_a file_b
hacker@dojo:~$ echo Look: file_??
Look: file_cc
```

Agora, pratique você mesmo!
Partindo do seu diretório home, mude de diretório para `/challenge`, mas utilize o caractere `?` no lugar de `c` e `l` no argumento passado para `cd`!
Assim que estiver lá, execute `/challenge/run` para pegar a flag!
