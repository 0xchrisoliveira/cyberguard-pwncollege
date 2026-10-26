Por padrão, as variáveis que você define em uma sessão de shell são estritamente locais àquele processo da shell.
Ou seja, outros comandos e programas que você executar a partir dela não as herdarão.
Você pode testar esse comportamento invocando uma nova shell filha dentro da sua própria shell:

```console
hacker@dojo:~$ VAR=1337
hacker@dojo:~$ echo "VAR is: $VAR"
VAR is: 1337
hacker@dojo:~$ sh
$ echo "VAR is: $VAR"
VAR is: 
```

Na saída acima, o prompt `$` pertence ao `sh`, uma implementação minimalista de shell invocada como processo *filho* da sua shell principal.
Observe que ela não recebeu o conteúdo da variável `VAR`!

Isso faz total sentido do ponto de vista de segurança e isolamento.
As variáveis da sua shell podem conter senhas, segredos ou dados arbitrários que não devem vazar para todos os utilitários que você roda, a menos que isso seja intencional.
Como sinalizar à shell que uma variável deve ser repassada?
Você *exporta* essa variável.
Ao exportar variáveis, elas passam a compor as *variáveis de ambiente* (*environment variables*) transmitidas aos processos filhos.
Veja um exemplo:

```console
hacker@dojo:~$ VAR=1337
hacker@dojo:~$ export VAR
hacker@dojo:~$ sh
$ echo "VAR is: $VAR"
VAR is: 1337
```

Aqui, o processo filho recebeu o valor de `VAR` e conseguiu imprimi-lo!
Você também pode condensar a declaração e exportação em uma única linha:

```console
hacker@dojo:~$ export VAR=1337
hacker@dojo:~$ sh
$ echo "VAR is: $VAR"
VAR is: 1337
```

Neste desafio, você deve executar `/challenge/run` com a variável `PWN` exportada e definida com o valor `COLLEGE`, e com a variável `COLLEGE` definida com o valor `PWN`, mas *não* exportada (ou seja, não herdada pelo `/challenge/run`).
Boa sorte!
