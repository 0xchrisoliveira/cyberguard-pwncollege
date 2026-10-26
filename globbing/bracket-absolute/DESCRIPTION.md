O globbing opera com base em caminhos (*paths*), portanto você pode expandir caminhos inteiros com argumentos baseados em glob.
Por exemplo:

```console
hacker@dojo:~$ touch file_a
hacker@dojo:~$ touch file_b
hacker@dojo:~$ touch file_c
hacker@dojo:~$ ls
file_a	file_b	file_c
hacker@dojo:~$ echo Look: /home/hacker/file_[ab]
Look: /home/hacker/file_a /home/hacker/file_b
```

Agora é a sua vez.
Mais uma vez, colocamos vários arquivos em `/challenge/files`.
A partir do seu diretório home, execute `/challenge/run` com um único argumento que use correspondência de colchetes (*bracket-glob*) para expandir nos caminhos absolutos dos arquivos `file_b`, `file_a`, `file_s` e `file_h`!
