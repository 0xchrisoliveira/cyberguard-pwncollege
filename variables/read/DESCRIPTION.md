Agora aprenderemos a capturar dados fornecidos pelo usuário (você).
Isso é feito com o comando embutido `read`, que lê dados da entrada padrão e os armazena em uma variável!

Veja um exemplo utilizando a opção `-p`, que permite definir um prompt de texto (caso contrário, seria difícil distinguir o que é entrada do usuário e o que é saída do sistema no exemplo a seguir):

```console
hacker@dojo:~$ read -p "INPUT: " MY_VARIABLE
INPUT: Hello!
hacker@dojo:~$ echo "You entered: $MY_VARIABLE"
You entered: Hello!
```

Lembre-se: o `read` lê dados a partir da sua entrada padrão (`stdin`)!
O primeiro `Hello!` acima foi *digitado* por você como entrada, e não exibido pelo comando.
Para deixar isso ainda mais claro, veja a anotação explícita de `INPUT` (entrada do usuário) e `OUTPUT` (saída exibida no terminal):

```console
 INPUT: hacker@dojo:~$ echo $MY_VARIABLE
OUTPUT:
 INPUT: hacker@dojo:~$ read MY_VARIABLE
 INPUT: Hello!
 INPUT: hacker@dojo:~$ echo "You entered: $MY_VARIABLE"
OUTPUT: You entered: Hello!
```

Neste desafio, seu objetivo é utilizar o comando `read` para atribuir o valor `COLLEGE` à variável `PWN`.
Boa sorte!
