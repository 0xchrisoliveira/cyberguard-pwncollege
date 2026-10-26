Em seguida, vamos abordar o `[]`.
Os colchetes são, essencialmente, uma forma restrita de `?`: em vez de corresponder a qualquer caractere, `[]` é um coringa para um subconjunto específico de caracteres potenciais, definidos dentro dos colchetes.
Por exemplo, `[pwn]` corresponderá ao caractere `p`, `w` ou `n`.
Veja um exemplo:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_c
hacker@dojo:~$ ls
file_a	file_b	file_c
hacker@dojo:~$ echo Look: file_[ab]
Look: file_a file_b
```

Experimente aqui!
Colocamos uma série de arquivos em `/challenge/files`.
Mude seu diretório de trabalho para `/challenge/files` e execute `/challenge/run` passando um único argumento com correspondência de colchetes (*bracket-glob*) que resulte em `file_b`, `file_a`, `file_s` e `file_h`!
