Você aprendeu a retomar processos em *primeiro plano* (*foreground*) com o comando `fg`.
Você também pode retomar processos em *segundo plano* (*background*) com o comando `bg`!
Isso permite que o processo continue rodando ativamente, ao mesmo tempo em que devolve o prompt da shell para você executar outros comandos enquanto ele trabalha.

O binário `run` deste nível quer ver outra cópia de si mesmo em execução, *não suspensa*, e utilizando o mesmo terminal.
Como fazer isso?
Use o terminal para iniciá-lo, suspenda-o com `Ctrl-Z`, envie-o para *segundo plano* com o comando `bg` e, então, inicie outra cópia enquanto a primeira continua rodando em background!

---

**CURIOSIDADE TÉCNICA:**
Se você tiver interesse em se aprofundar, veja como inspecionar as diferenças entre processos suspensos e processos rodando em segundo plano!
Permita-me demonstrar.
Primeiro, vamos suspender um comando `sleep`:

```console
hacker@dojo:~$ sleep 1337
^Z
[1]+  Stopped                 sleep 1337
hacker@dojo:~$
```

O processo `sleep` agora está *suspenso* em segundo plano.
Podemos verificar isso com o `ps`, ativando a coluna `stat` com a opção `-o`:

```console
hacker@dojo:~$ ps -o user,pid,stat,cmd
USER         PID STAT CMD
hacker       702 Ss   bash
hacker       762 T    sleep 1337
hacker       782 R+   ps -o user,pid,stat,cmd
hacker@dojo:~$ 
```

Viu aquele `T`?
Ele significa que o processo está suspenso (*stopped/traced*) devido ao nosso `Ctrl-Z`.
O `S` na coluna `STAT` do `bash` indica que ele está dormindo (*sleeping*), aguardando novas entradas de comandos.
O `R` na coluna do `ps` indica que ele está ativamente em execução (*running*), e o sinal `+` indica que ele está em primeiro plano (*foreground*)!

Veja o que acontece quando retomamos o `sleep` em segundo plano:

```console
hacker@dojo:~$ bg
[1]+ sleep 1337 &
hacker@dojo:~$ ps -o user,pid,stat,cmd
USER         PID STAT CMD
hacker       702 Ss   bash
hacker       762 S    sleep 1337
hacker      1224 R+   ps -o user,pid,stat,cmd
hacker@dojo:~$
```

Pronto!
O `sleep` agora possui o status `S`.
Ele está dormindo no sentido de aguardar o tempo passar, mas não está suspenso!
Ele também está em *segundo plano* (*background*) e, por isso, não tem mais o sinal `+`.
