Curiosamente, o comando `ls` não lista _todos_ os arquivos por padrão.
O Linux adota a convenção de que arquivos iniciados com um ponto `.` não são exibidos por padrão na listagem normal do `ls` nem em vários outros contextos.
Para visualizá-los com o `ls`, você precisa invocar o `ls` com o modificador (*flag*) `-a` (de *all*), desta forma:

```console
hacker@dojo:~$ touch pwn
hacker@dojo:~$ touch .college
hacker@dojo:~$ ls
pwn
hacker@dojo:~$ ls -a
.college	pwn
hacker@dojo:~$
```

Agora é a sua vez!
Vá e encontre a flag, que está escondida como um arquivo iniciado por ponto na raiz `/`.
