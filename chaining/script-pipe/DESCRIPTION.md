Vamos tentar algo um pouco mais engenhoso!
Você já canalizou saídas entre programas com o pipe `|`, mas até agora isso foi apenas entre a saída de um único comando e a entrada de outro comando.
Mas e se você quisesse enviar a saída combinada de vários programas para um único comando receptor?
Existem algumas formas de fazer isso, e exploraremos uma bastante direta aqui: redirecionar a saída do seu script!

Para a shell, o seu script é simplesmente mais um comando.
Isso significa que você pode redirecionar a entrada e a saída dele da mesma maneira que fazia com comandos no módulo de [Pipes e Redirecionamentos](/linux-luminarium/piping)!
Por exemplo, você pode gravar a saída do script em um arquivo:

```console
hacker@dojo:~$ cat script.sh
echo PWN
echo COLLEGE
hacker@dojo:~$ bash script.sh > output
hacker@dojo:~$ cat output
PWN
COLLEGE
hacker@dojo:~$
```

Todos os métodos de redirecionamento funcionam normalmente: `>` para stdout, `2>` para stderr, `<` para stdin, `>>` e `2>>` para modo anexação (*append*), `>&` para redirecionar entre descritores de arquivo e `|` para direcionar a saída em pipe para outro programa.

Neste nível, praticaremos o uso de pipe (`|`) da saída do seu script para outro programa.
Como abordaremos scripts diretamente executáveis mais adiante, execute seu script com `bash` ao canalizar a saída dele.
Como antes, crie um script `.sh` que invoque o comando `/challenge/pwn` seguido do comando `/challenge/college`, e redirecione a saída gerada pelo script através de um pipe para uma única invocação do comando `/challenge/solve`.
