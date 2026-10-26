Você já sabe redirecionar erros para um arquivo e sabe encadear saídas para outro programa usando pipes, como com `grep`.
Mas e se você quiser usar o `grep` diretamente nos fluxos de erro?

O operador `>` redireciona um descritor de arquivo para um arquivo em disco, e você já usou `2>` para redirecionar o FD 2 (saída de erro padrão).
Já o operador `|` redireciona *apenas a saída padrão* para outro programa, e não existe uma forma nativa como `2|`!
Ele redireciona *exclusivamente* o stdout (descritor de arquivo 1).

Felizmente, onde há uma shell, há uma solução!

A shell possui o operador `>&`, que redireciona um descritor de arquivo *para outro descritor de arquivo*.
Isso nos permite um processo em duas etapas para filtrar erros com `grep`: primeiro, redirecionamos a saída de erro padrão para a saída padrão (`2>&1`) e depois canalizamos com pipe (`|`) o fluxo combinado resultante diretamente para o `grep`!

Experimente agora!
Assim como no nível anterior, este desafio gerará um enorme volume de dados, mas desta vez na saída de erro padrão (`stderr`).
Utilize o `grep` filtrando esse fluxo para encontrar a flag!
