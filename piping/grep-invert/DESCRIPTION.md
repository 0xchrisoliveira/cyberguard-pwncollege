O comando `grep` possui uma opção extremamente útil: `-v` (*invert match* ou inversão de correspondência).
Enquanto o `grep` normal exibe apenas as linhas que CORRESPONDEM ao padrão, o `grep -v` exibe as linhas que NÃO correspondem ao padrão:

```console
hacker@dojo:~$ cat data.txt
hello hackers!
hello world!
hacker@dojo:~$ cat data.txt | grep -v world
hello hackers!
hacker@dojo:~$
```

Às vezes, a única maneira de isolar exatamente o dado desejado é filtrando e descartando tudo aquilo que você *não* quer.
Neste desafio, `/challenge/run` imprimirá a flag na saída padrão (`stdout`), mas junto a ela virão mais de 1000 flags falsas (contendo a palavra `DECOY` em alguma parte do texto) misturadas com a flag verdadeira.
Você precisará filtrar e eliminar todos os despistes (*decoys*) para revelar a flag real!

Use `grep -v` para descartar todas as linhas que contenham "DECOY" e obter a flag verdadeira!
