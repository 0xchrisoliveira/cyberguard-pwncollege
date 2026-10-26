Certo, as coisas quebram quando você esvazia a variável `PATH`.
Mas que tal fazermos algo _útil_ com o `PATH`?

Vamos explorar como poderíamos, por exemplo, adicionar um novo diretório de programas ao nosso repertório de comandos.
Lembre-se de que o `PATH` armazena uma lista de diretórios onde os comandos são procurados. Para comandos que estão em locais não padronizados, geralmente precisamos executá-los informando seus caminhos completos:

```console
hacker@dojo:~$ ls /home/hacker/scripts
goodscript	badscript	okayscript
hacker@dojo:~$ goodscript
bash: goodscript: command not found
hacker@dojo:~$ /home/hacker/scripts/goodscript
YEAH! This is the best script!
hacker@dojo:~$
```

Se você mantém scripts úteis que gostaria de executar digitando apenas seus nomes, ter que digitar o caminho inteiro é incômodo.
No entanto, ao adicionar ou substituir diretórios nessa lista, você pode disponibilizar esses programas para serem chamados apenas pelo nome!
Por exemplo:

```console
hacker@dojo:~$ PATH=/home/hacker/scripts
hacker@dojo:~$ goodscript
YEAH! This is the best script!
hacker@dojo:~$
```

Vamos praticar.
Neste nível, `/challenge/run` tentará executar o comando `win` apenas pelo nome, mas esse comando reside no diretório `/challenge/more_commands/`, que inicialmente não está no PATH.
O comando `win` é a _única_ coisa de que `/challenge/run` precisa, então você pode simplesmente sobrescrever o `PATH` com esse diretório específico.
Boa sorte!
