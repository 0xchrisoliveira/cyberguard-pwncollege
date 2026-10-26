Naturalmente, você não precisa iniciar um processo, suspendê-lo e depois enviá-lo para segundo plano: é possível iniciá-lo diretamente em segundo plano desde o primeiro instante!
É muito simples: basta adicionar o caractere `&` ao final do comando:

```console
hacker@dojo:~$ sleep 1337 &
[1] 1771
hacker@dojo:~$ ps -o user,pid,stat,cmd
USER         PID STAT CMD
hacker      1709 Ss   bash
hacker      1771 S    sleep 1337
hacker      1782 R+   ps -o user,pid,stat,cmd
hacker@dojo:~$ 
```

Aqui, o `sleep` já inicia rodando ativamente em segundo plano (*background*), sem jamais ter sido suspenso.
Agora é a sua vez de praticar!
Inicie `/challenge/run` em segundo plano para capturar a flag!
