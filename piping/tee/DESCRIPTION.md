Quando você canaliza dados de um comando para outro com pipe (`|`), você naturalmente deixa de vê-los na sua tela.
Isso nem sempre é o desejado: por exemplo, você pode querer inspecionar os dados conforme eles trafegam entre os comandos para depurar comportamentos inesperados (como por exemplo: "por que o segundo comando falhou???").

Felizmente, há uma solução perfeita!
O comando `tee`, cujo nome deriva da conexão em forma de "T" usada no encanamento hidráulico (*plumbing*), duplica os dados que fluem pelo pipe, enviando-os tanto para a saída padrão quanto para qualquer quantidade de arquivos informados na linha de comando.
Por exemplo:

```console
hacker@dojo:~$ echo hi | tee pwn college
hi
hacker@dojo:~$ cat pwn
hi
hacker@dojo:~$ cat college
hi
hacker@dojo:~$
```

Como visto acima, ao fornecer dois arquivos para o `tee`, obtivemos três cópias dos dados: uma na saída padrão (`stdout`), uma no arquivo `pwn` e outra no arquivo `college`.
Imagine como você pode usar isso para depurar pipelines com problemas:

```console
hacker@dojo:~$ command_1 | command_2
Command 2 failed!
hacker@dojo:~$ command_1 | tee cmd1_output | command_2
Command 2 failed!
hacker@dojo:~$ cat cmd1_output
Command 1 failed: must pass --succeed!
hacker@dojo:~$ command_1 --succeed | command_2
Commands succeeded!
```

Agora é a sua vez!
Neste desafio, a saída de `/challenge/pwn` deve ser conectada via pipe à entrada de `/challenge/college`, mas você precisará interceptar os dados para entender o que `/challenge/pwn` exige de você!
Use `tee` para duplicar a saída da sua primeira tentativa em um arquivo, inspecione as informações salvas e, em seguida, ajuste o seu comando com base no que descobriu.
