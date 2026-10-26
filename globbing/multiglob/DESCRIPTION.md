Até agora, você especificou apenas um padrão de glob de cada vez, mas é possível ir além!
O Bash suporta a expansão de múltiplos globs em uma única palavra.
Por exemplo:

```console
hacker@dojo:~$ cat /*fl*
pwn.college{YEAH}
hacker@dojo:~$
```

O que ocorre no exemplo acima é que a shell procura todos os arquivos em `/` que começam com *qualquer coisa* (inclusive nada), contêm `f` e `l`, e terminam com *qualquer coisa* (inclusive `ag`, formando `flag`).

Agora é a sua vez.
Colocamos alguns arquivos com nomes diversos em `/challenge/files`.
Use o `cd` para ir até lá e execute `/challenge/run`, passando um único argumento: uma palavra curta (com 3 caracteres ou menos) contendo dois coringas `*` que case com todas as palavras que contenham a letra `p`.
