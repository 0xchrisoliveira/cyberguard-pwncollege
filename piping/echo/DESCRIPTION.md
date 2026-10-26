Primeiramente, vamos explorar o redirecionamento da saída padrão (`stdout`) para arquivos.
Você pode realizar isso utilizando o caractere `>`, desta forma:

```console
hacker@dojo:~$ echo hi > asdf
```

Isso redirecionará a saída de `echo hi` (que seria `hi`) para o arquivo `asdf`.
Você pode então usar um comando como `cat` para inspecionar o conteúdo desse arquivo:

```console
hacker@dojo:~$ cat asdf
hi
```

Neste desafio, você deve usar esse redirecionamento de saída para escrever a palavra `PWN` (em letras maiúsculas) em um arquivo chamado `COLLEGE` (também em letras maiúsculas).
