O primeiro caractere coringa (*glob*) que aprenderemos é o `*`.
Quando a shell encontra um caractere `*` em qualquer argumento, ela o trata como um coringa (*wildcard*) e tenta substituir esse argumento por quaisquer arquivos que correspondam ao padrão.
É mais fácil demonstrar do que explicar:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_c
hacker@dojo:~$ ls
file_a	file_b	file_c
hacker@dojo:~$ echo Look: file_*
Look: file_a file_b file_c
```

Naturalmente, embora nesse caso o glob tenha resultado em múltiplos argumentos, ele também pode simplesmente corresponder a apenas um único arquivo:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ ls
file_a
hacker@dojo:~$ echo Look: file_*
Look: file_a
```

Quando nenhum arquivo corresponde ao padrão, por padrão a shell mantém o padrão inalterado:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ ls
file_a
hacker@dojo:~$ echo Look: nope_*
Look: nope_*
```

O `*` corresponde a qualquer parte do nome do arquivo, exceto pelo caractere `/` ou por um `.` no início do nome.
Por exemplo:

```console
hacker@dojo:~$ echo ONE: /ho*/*ck*
ONE: /home/hacker
hacker@dojo:~$ echo TWO: /*/hacker
TWO: /home/hacker
hacker@dojo:~$ echo THREE: ../*
THREE: ../hacker
```

Agora, pratique você mesmo!
Partindo do seu diretório home, mude de diretório para `/challenge`, mas use globbing para manter o argumento passado ao `cd` com no máximo quatro caracteres!
Assim que estiver lá, execute `/challenge/run` para obter a flag!
