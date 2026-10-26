Os seus blocos `if` até agora trataram casos específicos, mas e quando quisermos lidar com qualquer outro caso não previsto?
É exatamente aí que entra a cláusula `else`!

O bloco `else` é executado sempre que a condição do `if` for falsa:

```bash
if [ "$1" == "hello" ]
then
    echo "Hi there!"
else
    echo "I don't understand"
fi
```

Observe que o `else` não recebe uma condição de teste — ele captura qualquer entrada que não tenha correspondido anteriormente.
Ele também não utiliza a palavra-chave `then`.
Por fim, o `fi` vai após o bloco `else` para delimitar o término de toda a estrutura condicional!
O `else` é opcional: você não precisou dele no nível anterior e só deve usá-lo quando a lógica do seu script exigir um caso padrão.

Veja um exemplo prático:

```bash
if [ "$1" == "start" ]
then
    echo "Starting the service..."
else
    echo "Unknown command. Use 'start' to begin."
fi
```

Para este desafio, escreva um script em `/home/hacker/solve.sh` que:

- Receba um argumento
- Se o argumento for "pwn", imprima "college"
- Para qualquer outro valor de entrada, imprima "nope"

Exemplo:

```console
hacker@dojo:~$ bash /home/hacker/solve.sh pwn
college
hacker@dojo:~$ bash /home/hacker/solve.sh hack
nope
hacker@dojo:~$ bash /home/hacker/solve.sh anything
nope
hacker@dojo:~$
```

Assim que o script funcionar conforme esperado, execute `/challenge/run` para capturar a flag!
