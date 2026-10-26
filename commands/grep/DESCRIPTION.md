Às vezes, os arquivos que você deseja ler com o `cat` são grandes demais.
Felizmente, temos o comando `grep` para buscar exatamente o conteúdo de que precisamos!
Aprenderemos a usá-lo neste desafio.

Existem muitas formas de usar o `grep`, e aprenderemos uma delas aqui:

```console
hacker@dojo:~$ grep SEARCH_STRING /path/to/file
```

Invocado dessa forma, o `grep` buscará no arquivo linhas de texto contendo `SEARCH_STRING` e as exibirá no console.

Neste desafio, foram inseridas centenas de milhares de linhas de texto no arquivo `/challenge/data.txt`.
Use o `grep` nele para encontrar a flag!

DICA: A flag sempre começa com o texto `pwn.college`.
