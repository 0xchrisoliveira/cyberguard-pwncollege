Agora que você consegue usar argumentos em scripts, vamos torná-los mais inteligentes com lógica condicional!

No bash, você pode usar blocos `if` para tomar decisões:

```bash
if [ "$1" == "ping" ]
then
    echo "pong"
fi
```

O código acima significa: `se o primeiro argumento for igual a "ping", imprima "pong"`.
A sintaxe do bash é bastante rigorosa por algumas razões:
Primeiro, você *deve* obrigatoriamente incluir espaços após o `if`, após o `[` de abertura e antes do `]` de fechamento.
Segundo, `if`, `then` e `fi` devem estar em linhas separadas (ou separados por ponto e vírgula); você não pode juntá-los na mesma linha sem `;`.
Além disso, há uma peculiaridade clássica: em vez de `endif` ou algo parecido, o fechamento do bloco `if` é `fi` (`if` escrito ao contrário). É uma idiossincrasia do shell que você precisa memorizar.

Para este desafio, escreva um script em `/home/hacker/solve.sh` que:

- Receba um argumento
- Se o argumento for "pwn", imprima "college"
- Para qualquer outra entrada, não imprima nada

Exemplo:

```console
hacker@dojo:~$ bash /home/hacker/solve.sh pwn
college
hacker@dojo:~$ bash /home/hacker/solve.sh foo
hacker@dojo:~$
```
Assim que o seu script estiver funcionando corretamente, execute `/challenge/run` para capturar a sua flag!

----
**NOTA:**
Ficou curioso sobre o que mais você pode testar em uma condição além da igualdade de strings?
Descubra todas as comparações possíveis executando `help test` no seu terminal!
