Pobre Zardus; você já o invadiu bastante.
Mas ele aprendeu a lição e protegeu seu diretório inicial!
Fim de jogo?

Ainda não!
Uma das coisas que as pessoas frequentemente não consideram quando existem múltiplas contas no mesmo computador é que tipo de informação a _execução dos seus comandos_ pode vazar.
Lembre-se: quando você executa `ps aux`, você obtém:

```
hacker@dojo:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
hacker         1  0.0  0.0   1128     4 ?        Ss   05:34   0:00 /sbin/docker-init -- /bin/sleep 6h
hacker         7  0.0  0.0   2736   580 ?        S    05:34   0:00 /bin/sleep 6h
hacker       102  0.4  0.0 723944 64660 ?        Sl   05:34   0:00 /usr/lib/code-server/lib/node /usr/lib/code-serve
hacker       138  3.3  0.0 968792 106272 ?       Sl   05:34   0:07 /usr/lib/code-server/lib/node /usr/lib/code-serve
hacker       287  0.0  0.0 717648 53136 ?        Sl   05:34   0:00 /usr/lib/code-server/lib/node /usr/lib/code-serve
hacker       318  3.3  0.0 977472 98256 ?        Sl   05:34   0:06 /usr/lib/code-server/lib/node --dns-result-order=
hacker       554  0.4  0.0 650560 55360 ?        Rl   05:35   0:00 /usr/lib/code-server/lib/node /usr/lib/code-serve
hacker       571  0.0  0.0   4600  4032 pts/0    Ss   05:35   0:00 /usr/bin/bash --init-file /usr/lib/code-server/li
hacker      1172  0.0  0.0   5892  2924 pts/0    R+   05:38   0:00 ps aux
hacker@dojo:~$
```

Mas o que aconteceria se um dos argumentos passados para esses comandos contivesse dados sensíveis, como a flag ou uma senha?
Isso acontece na vida real, e usuários mal-intencionados que compartilham a mesma máquina (ou que consigam listar processos nela de alguma forma) podem roubar esses dados e usá-los!

É isso o que este desafio explora.
Zardus está utilizando um script de automação, passando a senha da sua conta como argumento na linha de comando.
Além disso, Zardus tem permissão para usar `sudo` (e, portanto, pode executar `sudo cat /flag`!).
Roube a senha, faça login na conta do Zardus (relembre o comando `su` do módulo [Desvendando Usuários](/linux-luminarium/users)) e capture a flag!
