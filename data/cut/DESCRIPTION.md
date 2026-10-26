Às vezes, você precisa extrair colunas de dados específicas, como a primeira coluna, a terceira coluna ou a 42ª coluna.
Para isso, existe o comando `cut`.

Por exemplo, imagine que você tenha o seguinte arquivo de dados:

```console
hacker@dojo:~$ cat scores.txt
hacker 78 99 67
root 92 43 89
hacker@dojo:~$
```

Você pode usar o `cut` para isolar colunas específicas:

```console
hacker@dojo:~$ cut -d " " -f 1 scores.txt
hacker
root
hacker@dojo:~$ cut -d " " -f 2 scores.txt
78
92
hacker@dojo:~$ cut -d " " -f 3 scores.txt
99
43
hacker@dojo:~$
```

O parâmetro `-d` especifica o *delimitador* das colunas (como os campos são separados entre si).
Neste exemplo, é o caractere de espaço em branco.
Naturalmente, ele deve estar entre aspas para que a shell compreenda que o espaço é o valor do argumento, e não um separador sintático de argumentos!
Já o parâmetro `-f` especifica o número do campo (*field*) ou coluna a ser extraída (iniciando em 1).

Neste desafio, o programa `/challenge/run` fornecerá várias linhas contendo números aleatórios e caracteres individuais (caracteres pertencentes à flag) dispostos em colunas.
Use o comando `cut` para isolar a coluna com os caracteres da flag e, em seguida, canalize a saída para `tr -d "\n"` (como no nível anterior!) para reuni-los em uma única linha contínua.
Sua solução terá a estrutura de `/challenge/run | cut ??? | tr ???`, substituindo os `???` pelos parâmetros corretos.
