No nível anterior, você executou `cat flag` para ler a flag diretamente do seu diretório home!
Naturalmente, você também pode especificar os argumentos do `cat` como caminhos absolutos:

```console
hacker@dojo:~$ cat /path/to/file
Hello Hackers!
```

Neste desafio, a flag não foi copiada para o seu diretório home, mas suas permissões foram liberadas para leitura.
Você pode lê-la com `cat` informando seu caminho absoluto: `/flag`.

----
**CURIOSIDADE:**
`/flag` é onde a flag _sempre_ fica no pwn.college, mas, ao contrário deste desafio, normalmente você não tem permissão para acessar esse arquivo diretamente.
