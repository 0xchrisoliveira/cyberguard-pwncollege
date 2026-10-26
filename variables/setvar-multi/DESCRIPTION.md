Neste nível, você aprenderá sobre o uso de aspas (*quoting*).
Os espaços em branco possuem um significado sintático especial na shell, e não podem ser usados de maneira avulsa.
Relembre como definimos uma variável:

```console
hacker@dojo:~$ VAR=1337
```

Isso atribui o valor `1337` à variável `VAR`, mas e se você quisesse definir o valor como `1337 SAUCE`?
Você poderia ser tentado a digitar:

```console
hacker@dojo:~$ VAR=1337 SAUCE
```

Isso parece intuitivo, mas não funciona, por motivos análogos à regra de não colocar espaços ao redor de `=`.
Quando a shell encontra um espaço não delimitado, ela encerra a atribuição da variável e interpreta o próximo termo (`SAUCE`, neste caso) como sendo um comando a executar.
Para definir `VAR` com o valor `1337 SAUCE`, você precisa delimitar a expressão com aspas (*quotes*):

```console
hacker@dojo:~$ VAR="1337 SAUCE"
```

Dessa forma, a shell interpreta `1337 SAUCE` como um único argumento/token textual, atribuindo com sucesso o valor completo para `VAR`.
Neste nível, você precisa definir a variável `PWN` com o valor `COLLEGE YEAH`.
Boa sorte!
