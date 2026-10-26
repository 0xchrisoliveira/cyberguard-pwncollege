Quando você digita o nome de um comando, _algo_ dentro de um dos vários diretórios listados na sua variável `$PATH` é o que realmente é executado (a menos, é claro, que o comando seja um comando embutido (*builtin*) da shell!).
Mas _qual_ arquivo exatamente?
Você pode descobrir isso com o apropriadamente chamado comando `which`:

```console
hacker@dojo:~$ which cat
/bin/cat
hacker@dojo:~$ /bin/cat /flag
YEAH
hacker@dojo:~$
```

Espelhando exatamente o que a shell faz ao procurar comandos, o `which` examina cada diretório do `$PATH` em ordem e imprime o primeiro arquivo encontrado cujo nome corresponda ao argumento fornecido.

Neste desafio, adicionamos um comando `win` em algum lugar do seu `$PATH`, mas executá-lo não lhe dará a flag diretamente.
Em vez disso, ele está localizado no mesmo diretório de um arquivo `flag` com permissão de leitura para você!
Você deve encontrar o `win` (usando o comando `which`) e usar `cat` para ler a flag daquele diretório!
