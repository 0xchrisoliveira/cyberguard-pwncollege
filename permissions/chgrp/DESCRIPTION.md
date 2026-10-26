Compartilhar é se importar, e o compartilhamento está no cerne do design do Linux.
Os arquivos possuem tanto um *usuário* dono quanto um *grupo* dono.
Um grupo pode conter múltiplos usuários, e um usuário pode ser membro de vários grupos simultaneamente.

Você pode conferir de quais grupos faz parte utilizando o comando `id`:

```console
hacker@dojo:~$ id
uid=1000(hacker) gid=1000(hacker) groups=1000(hacker)
hacker@dojo:~$
```

Aqui, o usuário `hacker` faz parte *apenas* do grupo `hacker`.
O caso de uso mais comum para grupos é controlar o acesso a diferentes recursos do sistema operacional.
Por exemplo, o "Modo Privilegiado" (*Privileged Mode*) no pwn.college concede a você acesso root para facilitar depurações e testes.
Isso é feito adicionando um grupo extra ao seu usuário quando você inicia no modo privilegiado:

```console
hacker@dojo:~$ id
uid=1000(hacker) gid=1000(hacker) groups=1000(hacker),27(sudo)
hacker@dojo:~$
```

Um usuário comum em um computador desktop Linux típico participa de *muitos* grupos.
Por exemplo, veja a saída no computador do Zardus:

```console
zardus@yourcomputer:~$ id
uid=1000(zardus) gid=1000(zardus) groups=1000(zardus),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev),114(bluetooth),117(lpadmin),120(scanner),995(docker)
zardus@yourcomputer:~$
```

Todos esses grupos dão a Zardus a capacidade de ler CDs e disquetes (quem ainda usa isso?), administrar o sistema, reproduzir áudio, desenhar na tela de vídeo, usar bluetooth e assim por diante.
Geralmente, esse controle de acesso ocorre por meio da propriedade de grupo no sistema de arquivos!
Por exemplo, a saída gráfica de vídeo pode ser controlada através do arquivo especial de dispositivo `/dev/fb0`:

```console
zardus@yourcomputer:~$ ls -l /dev/fb0
crw-rw---- 1 root video 29, 0 Jun 30 23:42 /dev/fb0
zardus@yourcomputer:~$
```

Esse arquivo é um *arquivo de dispositivo* especial (o tipo `c` indica que é um "character device"), e interagir com ele altera a exibição da tela (em vez de alterar dados gravados no disco, como em um arquivo convencional!).
A conta de usuário de Zardus pode interagir com ele porque o grupo dono do arquivo é `video`, e Zardus pertence ao grupo `video`.

Mas não temos essa facilidade com o arquivo `/flag` no dojo!
Veja o seguinte cenário:

```console
hacker@dojo:~$ id
uid=1000(hacker) gid=1000(hacker) groups=1000(hacker)
hacker@dojo:~$ ls -l /flag
-r--r----- 1 root root ... /flag
hacker@dojo:~$ cat /flag
cat: /flag: Permission denied
hacker@dojo:~$
```

Aqui, o arquivo da flag pertence ao usuário `root` e ao grupo `root`, e o usuário `hacker` não é o usuário `root` nem pertence ao grupo `root`. Logo, o acesso é bloqueado.
Felizmente, a propriedade de grupo de um arquivo pode ser modificada com o comando `chgrp` (**ch**ange **gr**ou**p**)!
A menos que você seja o dono do arquivo *e* membro do novo grupo, essa operação normalmente requer acesso de `root`. Então, vamos testar como `root`:

```console
root@dojo:~# mkdir pwn_directory
root@dojo:~# touch college_file
root@dojo:~# ls -l
total 4
-rw-r--r-- 1 root root    0 May 22 13:42 college_file
drwxr-xr-x 2 root root 4096 May 22 13:42 pwn_directory
root@dojo:~# chgrp hacker college_file
root@dojo:~# ls -l
total 4
-rw-r--r-- 1 root hacker    0 May 22 13:42 college_file
drwxr-xr-x 2 root root   4096 May 22 13:42 pwn_directory
root@dojo:~#
```

Neste nível, configurei a flag para que ela possa ser lida por qualquer grupo que seja seu proprietário, mas esse grupo atualmente é o `root`.
Felizmente, também permiti que você invoque o comando `chgrp` como o usuário `hacker`!
Mude a propriedade de grupo do arquivo da flag e leia a flag!
