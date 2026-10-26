Às vezes, processos com comportamento indevido podem interferir no seu trabalho.
Esses processos precisam ser finalizados...

Neste desafio, há um processo de despiste (*decoy*) monopolizando um recurso crítico: um named pipe (FIFO) em `/tmp/flag_fifo` no qual (assim como no desafio de FIFO de [Practicing Piping](/linux-luminarium/piping)) `/challenge/run` quer gravar a sua flag.
Você precisará encerrar (`kill`) esse processo.

Seu fluxo geral de trabalho deve ser:

1. Verificar quais processos estão em execução.
2. Encontrar `/challenge/decoy` na lista e identificar o seu ID de processo (PID).
3. Finalizá-lo com `kill`.
4. Executar `/challenge/run` para obter a flag sem ser inundado por mensagens falsas (você não precisa redirecionar a saída dele; ele gravará no FIFO por conta própria).

Boa sorte!

----
**NOTA:**
Você pode ver algumas flags falsas aparecerem mesmo após finalizar o processo despiste.
Isso acontece porque os pipes no Linux têm *buffer* (*buffered*): conceitualmente, eles possuem uma extensão pela qual os dados fluem, e você pode ter finalizado o processo enquanto ainda havia dados trafegando no pipe.
Esses dados, já tendo entrado no pipe, seguirão até a outra ponta (o seu `cat`).
Se esperar um segundo, você verá que os despistes cessarão, e então você poderá rodar `/challenge/run` e vencer!
