Os arquivos estão por toda parte.
Assim como embalagens de doces, eventualmente haverá arquivos demais acumulados.
Neste nível, aprenderemos a fazer uma faxina!

No Linux, você **r**e**m**ove arquivos usando o comando `rm` (*remove*), desta forma:

```console
hacker@dojo:~$ touch PWN
hacker@dojo:~$ touch COLLEGE
hacker@dojo:~$ ls
COLLEGE     PWN
hacker@dojo:~$ rm PWN
hacker@dojo:~$ ls
COLLEGE
hacker@dojo:~$
```

Vamos praticar.
Este desafio criará um arquivo chamado `delete_me` no seu diretório home!
Exclua-o e, em seguida, execute `/challenge/check`, que verificará se você o removeu corretamente e então entregará a sua flag!
