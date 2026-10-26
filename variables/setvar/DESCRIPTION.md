Naturalmente, além de ler os valores armazenados em variáveis, você também pode gravar valores nelas.
Isso é feito, como em muitas outras linguagens de programação, utilizando o operador `=`.
Para atribuir o valor `1337` à variável `VAR`, você utiliza:

```console
hacker@dojo:~$ VAR=1337
```

Observe que não deve haver espaços ao redor do sinal de `=`!
Se você colocar espaços (por exemplo, `VAR = 1337`), a shell não reconhecerá a operação como uma atribuição de variável e tentará, em vez disso, executar um comando chamado `VAR` (que não existe).

Note também que aqui usamos `VAR` e *não* `$VAR`: o `$` só é prefixado quando queremos *acessar* o conteúdo da variável.
Nos termos da shell, o uso de `$` dispara o que chamamos de *expansão de variável* (*variable expansion*), sendo inclusive a origem de inúmeras vulnerabilidades de segurança comuns (se tiver interesse nisso, confira o dojo Art of the Shell assim que estiver confortável com a linha de comando!).

Após definir as variáveis, você pode acessá-las com os métodos aprendidos anteriormente:

```console
hacker@dojo:~$ echo $VAR
1337
```

Para resolver este nível, você deve definir a variável `PWN` com o valor `COLLEGE`.
Atenção: tanto os nomes quanto os valores das variáveis diferenciam maiúsculas de minúsculas (*case-sensitive*)!
`PWN` não é o mesmo que `pwn`, e `COLLEGE` não é o mesmo que `College`.
