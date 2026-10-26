Às vezes, você quer filtrar e excluir arquivos em um padrão glob!
Felizmente, o `[]` ajuda exatamente nisso.
Se o primeiro caractere dentro dos colchetes for `!` ou (em versões mais recentes do bash) `^`, o glob é invertido, e essa instância de colchetes corresponderá a caracteres que *não* estejam listados.
Por exemplo:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_c
hacker@dojo:~$ ls
file_a	file_b	file_c
hacker@dojo:~$ echo Look: file_[!ab]
Look: file_c
hacker@dojo:~$ echo Look: file_[^ab]
Look: file_c
hacker@dojo:~$ echo Look: file_[ab]
Look: file_a file_b
```

Com esse conhecimento em mãos, vá até `/challenge/files` e execute `/challenge/run` passando como argumento todos os arquivos que não comecem com `p`, `w` ou `n`!

**NOTA:** O caractere `!` possui um significado especial diferente no bash quando não é o primeiro caractere dentro de um glob `[]`, portanto tenha isso em mente caso as coisas comecem a se comportar de forma estranha! O `^` não sofre desse problema, mas pode não ser compatível com shells mais antigas.
