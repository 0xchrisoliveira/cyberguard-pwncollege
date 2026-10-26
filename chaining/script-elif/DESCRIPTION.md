Você aprendeu a usar uma única instrução `if` para avaliar uma condição.
Mas e quando você precisa avaliar múltiplas condições em sequência?
Você pode usar a cláusula `elif` (abreviação de `else if`):

```bash
if [ "$1" == "one" ]
then
    echo "1"
elif [ "$1" == "two" ]
then
    echo "2"
elif [ "$1" == "three" ]
then
    echo "3"
else
    echo "unknown"
fi
```

Observe que você *precisa* colocar um `then` após cada `elif`, exatamente como no `if`.
E como antes, o `else` no final captura qualquer valor que não tenha correspondido a nenhuma das opções anteriores.

Para este desafio, escreva um script em `/home/hacker/solve.sh` que:

- Receba um argumento
- Se o argumento for "hack", imprima "the planet"
- Se o argumento for "pwn", imprima "college"  
- Se o argumento for "learn", imprima "linux"
- Para qualquer outro valor de entrada, imprima "unknown"

Exemplo:

```console
hacker@dojo:~$ bash /home/hacker/solve.sh hack
the planet
hacker@dojo:~$ bash /home/hacker/solve.sh pwn
college
hacker@dojo:~$ bash /home/hacker/solve.sh learn
linux
hacker@dojo:~$ bash /home/hacker/solve.sh foo
unknown
hacker@dojo:~$
```

Assim que seu script estiver funcionando corretamente, execute `/challenge/run` para capturar a sua flag!

----
**NOTA:**
Ao criar seu script, certifique-se de respeitar rigorosamente os espaços apresentados nos exemplos.
Diferente de muitas outras linguagens, o bash exige que `[` e `]` estejam separados de outros caracteres por espaços em branco, caso contrário a shell não conseguirá interpretar a condição.
