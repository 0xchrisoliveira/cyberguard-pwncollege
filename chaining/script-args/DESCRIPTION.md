Você aprendeu a criar shell scripts, mas até agora eles eram apenas listas fixas de comandos.
Os scripts tornam-se infinitamente mais poderosos quando conseguem receber argumentos dinâmicos!
Isso se parece com:

```console
hacker@dojo:~$ bash myscript.sh hello world
```

O script pode acessar esses argumentos utilizando variáveis posicionais especiais:
- `$1` contém o primeiro argumento ("hello")
- `$2` contém o segundo argumento ("world")
- `$3` conteria o terceiro argumento (se houvesse um)
- ...e assim por diante

Aqui está um exemplo simples:
```bash
hacker@dojo:~$ cat myscript.sh
#!/bin/bash
echo "First argument: $1"
echo "Second argument: $2"
hacker@dojo:~$ bash myscript.sh hello world
First argument: hello
Second argument: world
hacker@dojo:~$
```

Para este desafio, você precisa escrever um script em `/home/hacker/solve.sh` que:
1. Receba dois argumentos
2. Imprima esses argumentos em ORDEM INVERSA (o segundo argumento primeiro, seguido do primeiro argumento)

Por exemplo:
```console
hacker@dojo:~$ bash /home/hacker/solve.sh pwn college
college pwn
hacker@dojo:~$
```

Assim que seu script estiver funcionando corretamente, execute `/challenge/run` para capturar a sua flag!
