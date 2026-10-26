Certos comandos, em vez de serem programas externos com páginas man e opções de ajuda próprias, são embutidos diretamente na própria shell.
Eles são chamados de comandos internos ou *builtins*.
Os builtins são executados exatamente como qualquer outro comando, mas a shell os processa internamente em vez de lançar um novo programa no sistema.
Você pode obter uma lista completa dos comandos builtins da shell executando o *builtin* `help`, desta forma:

```console
hacker@dojo:~$ help
```

Você pode obter ajuda sobre um comando específico passando-o como argumento para o builtin `help`.
Vejamos a documentação de um comando interno que já utilizamos antes, o `cd`!

```console
hacker@dojo:~$ help cd
cd: cd [-L|[-P [-e]] [-@]] [dir]
    Change the shell working directory.
    
    Change the current directory to DIR.  The default DIR is the value of the
    HOME shell variable.
...
```

Excelente informação!
Neste desafio, praticaremos o uso do `help` para consultar a documentação de comandos builtins.
O comando `challenge` deste nível é um comando interno da shell (*builtin*), e não um arquivo executável isolado.
Como antes, você precisará consultar a sua ajuda para descobrir o valor secreto que deve ser passado a ele!
