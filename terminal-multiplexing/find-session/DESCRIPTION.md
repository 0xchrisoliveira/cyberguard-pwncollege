Hora de bancar o detetive com o screen!

Se você se tornar um usuário frequente do screen, inevitavelmente acabará com múltiplas sessões em execução.
Como descobrir qual é a sessão certa para se reconectar?

Bem, nós podemos listá-las:

```console
hacker@dojo:~$ screen -ls
There are screens on:
        23847.mysession   (Detached)
        23851.goodwork    (Detached)
        23855.morework    (Detached)
3 Sockets in /run/screen/S-hacker.
```

Os identificadores das sessões são compostos pelo PID de cada processo do screen, um ponto e o nome da sessão.
Para se conectar a uma específica, utilize o nome ou o PID dela passando-o como argumento para `screen -r`.

```console
hacker@dojo:~$ screen -r goodwork
```

Neste desafio, criamos três sessões do screen para você.
Uma delas contém a flag.
As outras duas são despistes!

Você precisará verificar cada uma até encontrar a correta.
Não se esqueça de desconectar (`Ctrl-A d`) antes de tentar a próxima sessão!
