Agora, vamos reunir todo o seu conhecimento.
Você deve dominar a tarefa definitiva de piping: redirecionar o `stdout` para um programa e o `stderr` para outro programa simultaneamente.

O desafio aqui é que o operador `|` conecta apenas o `stdout` do comando à esquerda com o `stdin` do comando à direita.
Você já usou `2>&1` para redirecionar stderr para stdout e enviar via pipe, mas isso mistura os dois fluxos em um só.
Como mantê-los separados e direcionados a processos distintos?

Você precisará combinar seu conhecimento de `>()`, `2>` e `|`.
A forma exata de estruturar essa linha é um quebra-cabeça que deixamos com você.

Neste desafio, você tem:

- `/challenge/hack`: este binário produz dados tanto no `stdout` quanto no `stderr`
- `/challenge/the`: você deve redirecionar o `stderr` do `hack` para a entrada deste programa
- `/challenge/planet`: você deve redirecionar o `stdout` do `hack` para a entrada deste programa

Vá em busca da flag!

----
**DESAFIO BÔNUS:** Para um nível extra de maestria, encontre uma solução que funcione sem utilizar o operador `|`.
