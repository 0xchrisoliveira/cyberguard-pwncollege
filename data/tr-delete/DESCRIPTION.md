O comando `tr` também pode traduzir caracteres para nada (ou seja, *deletá-los* do fluxo).
Isso é feito com a opção `-d` (*delete*) acompanhada dos caracteres que devem ser eliminados:

```console
hacker@dojo:~$ echo PAWN | tr -d A
PWN
hacker@dojo:~$
```

Muito direto e simples!
Agora experimente você mesmo.
Na saída do binário `/challenge/run`, colocamos alguns caracteres intrusos de ruído (especificamente: `^` e `%`) intercalados entre os caracteres da flag.
Utilize o `tr -d` para eliminá-los!
