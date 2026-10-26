Até agora, você lidou principalmente com permissões de *leitura* (*read*).
Isso faz todo sentido, já que você estava tornando o arquivo `/flag` legível para ler o seu conteúdo.
Neste nível, exploraremos as permissões de *execução* (*execute*).

Quando você executa um programa, como `/challenge/run`, o Linux só permite a execução efetiva se o seu usuário tiver permissão de execução naquele arquivo.
Veja:

```console
hacker@dojo:~$ ls -l /challenge/run
-rwxr-xr-x 1 root root    0 May 22 13:42 /challenge/run
hacker@dojo:~$ /challenge/run
Successfully ran the challenge!
hacker@dojo:~$
```

Neste caso, `/challenge/run` executa porque ele tem permissão de execução para o usuário `hacker`.
Como o arquivo pertence ao usuário `root` e ao grupo `root`, isso exige que o bit de execução esteja ativo nas permissões de "outros" (*other*).
Se removermos essa permissão, a execução falhará:

```console
hacker@dojo:~$ chmod o-x /challenge/run
hacker@dojo:~$ ls -l /challenge/run
-rwxr-xr-- 1 root root    0 May 22 13:42 /challenge/run
hacker@dojo:~$ /challenge/run
bash: /challenge/run: Permission denied
hacker@dojo:~$
```

Neste desafio, o programa `/challenge/run` entregará a flag para você, mas antes você precisa torná-lo executável!
Lembre-se do que aprendeu sobre o `chmod` e faça `/challenge/run` revelar a sua flag!
