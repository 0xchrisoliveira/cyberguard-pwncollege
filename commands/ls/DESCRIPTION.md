Até agora, nós dissemos com quais arquivos você deveria interagir.
No entanto, os diretórios podem conter muitos arquivos (e outros subdiretórios) em seu interior, e nem sempre estaremos aqui para revelar seus nomes.
Você precisará aprender a **l**i**s**tar o conteúdo deles usando o comando `ls` (*list*)!

O comando `ls` lista os arquivos em todos os diretórios passados a ele como argumentos, ou no diretório atual de trabalho se nenhum argumento for fornecido.
Observe:

```console
hacker@dojo:~$ ls /challenge
run
hacker@dojo:~$ ls
Desktop    Downloads  Pictures  Templates
Documents  Music      Public    Videos
hacker@dojo:~$ ls /home/hacker
Desktop    Downloads  Pictures  Templates
Documents  Music      Public    Videos
hacker@dojo:~$
```

Neste desafio, renomeamos `/challenge/run` para um nome aleatório!
Liste os arquivos em `/challenge` para descobri-lo.
Em seguida, invoque o caminho absoluto descoberto para obter a flag.
