Um tipo muito comum de caractere que precisamos remover são as quebras de linha (*line separators*).
Isso ocorre quando temos um fluxo distribuído em várias linhas e queremos uni-lo em uma única linha contínua para processamento posterior.
Você pode representar quebras de linha quase como qualquer outro caractere, usando o caractere de escape:

```console
hacker@dojo:~$ echo "hello_world!" | tr _ "\n"
hello
world!
hacker@dojo:~$
```

Aqui, a barra invertida (`\`) indica que o caractere que a segue representa um caractere especial que não é fácil de digitar diretamente no terminal.
Afinal, a quebra de linha é difícil de digitar diretamente porque pressionar a tecla `Enter` normalmente dispara a execução do comando.
A sequência `\n` é a representação dessa nova linha, e ela *deve* estar entre aspas para que a shell não a interprete de maneira antecipada e a repasse intacta ao `tr`.

Agora, vamos combinar isso com a exclusão de caracteres.
Neste desafio, injetamos uma série de quebras de linha no meio da flag.
Remova todas elas utilizando a opção `-d` do `tr` junto com a especificação da quebra de linha escapada!

----
**Curiosidade:**
Quer substituir literalmente o próprio caractere de barra invertida (`\`)?
Como a `\` é o caractere de escape, você precisa escapar a própria barra!
A sequência `\\` será interpretada como uma barra invertida literal pelo `tr`.
Isso não é necessário para este desafio em particular, mas é uma dica valiosa para a vida!
