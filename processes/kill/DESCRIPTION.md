Você já iniciou processos e já visualizou processos; agora você aprenderá a *encerrar* processos!
No Linux, isso é feito através do comando de nome expressivo: `kill`.
Com as opções padrão (que é o que abordaremos neste nível), o `kill` encerra um processo de forma que ele ainda tenha a oportunidade de salvar pendências e finalizar adequadamente antes de deixar de existir (enviando o sinal SIGTERM).

Digamos que você tenha um incômodo processo `sleep` (`sleep` é um utilitário que simplesmente fica esperando durante a quantidade de segundos especificada na linha de comando; neste caso, 1337 segundos) iniciado em outro terminal:

```console
hacker@dojo:~$ sleep 1337
```

Como nos livramos dele?
Você usa o `kill` para finalizá-lo, passando o identificador do processo (o `PID` obtido via `ps`) como argumento:

```console
hacker@dojo:~$ ps -e | grep sleep
 342 pts/0    00:00:00 sleep
hacker@dojo:~$ kill 342
hacker@dojo:~$ ps -e | grep sleep
hacker@dojo:~$
```

Agora é hora de finalizar seu primeiro processo!
Neste desafio, `/challenge/run` se recusará a rodar enquanto `/challenge/dont_run` estiver em execução!
Você precisa encontrar o processo de `dont_run` e usar o `kill` nele.
Se falhar, o `pwn.college` negará qualquer conhecimento sobre sua missão.
Boa sorte!
