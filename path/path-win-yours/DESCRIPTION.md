Relembre o exemplo do nível anterior:

```console
hacker@dojo:~$ ls /home/hacker/scripts
goodscript	badscript	okayscript
hacker@dojo:~$ PATH=/home/hacker/scripts
hacker@dojo:~$ goodscript
YEAH! This is the best script!
hacker@dojo:~$
```

O que vemos aqui, é claro, é o `hacker` tornando a shell mais útil para si mesmo ao trazer seus próprios comandos para o jogo.
Com o tempo, você construirá seu próprio conjunto de ferramentas elegantes.
Vamos começar com o `win`!

Anteriormente, o comando `win` executado por `/challenge/run` estava armazenado em `/challenge/more_commands`.
Desta vez, o comando `win` simplesmente não existe!
Relembre o último nível de [Encadeando Comandos](/linux-luminarium/chaining), crie um shell script chamado `win`, adicione a localização dele ao `PATH` e permita que `/challenge/run` o encontre!

----
**Dica:**
`/challenge/run` roda como `root` e chamará `win`. Logo, o seu script `win` pode simplesmente ler o arquivo da flag com `cat`.
Mais uma vez, o comando `win` é a _única_ coisa necessária para o `/challenge/run`, então você pode apenas sobrescrever o `PATH` com esse diretório.
Mas lembre-se: se fizer isso, o seu comando `win` não conseguirá encontrar o `cat` pelo nome.

Você tem três opções para contornar isso:

1. Descobrir onde o programa `cat` está no sistema de arquivos. Ele _obrigatoriamente_ está em um dos diretórios presentes na variável `PATH`, então você pode exibir o conteúdo dessa variável (consulte [Variáveis de Shell](/linux-luminarium/variables) para relembrar como!), examinar os diretórios listados (lembre-se de que as entradas são separadas por `:`), encontrar qual deles contém o `cat` e chamar `cat` por seu caminho absoluto.
2. Definir um `PATH` contendo os diretórios originais _mais_ uma nova entrada apontando para onde você criou o `win`.
3. Usar `read` (novamente, consulte [Variáveis de Shell](/linux-luminarium/variables)) para ler `/flag`. Como `read` é um recurso embutido (*builtin*) do `bash`, ele não é afetado por alterações no `PATH`.

Agora vá em frente e alcance o `win`!
