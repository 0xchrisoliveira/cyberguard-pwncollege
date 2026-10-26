Além de redirecionar a saída de `echo`, você pode, evidentemente, redirecionar a saída de qualquer comando.
Neste nível, `/challenge/run` entregará a flag novamente, mas *apenas* se você redirecionar a sua saída para o arquivo `myflag`.
Sua flag será gravada diretamente no arquivo `myflag`!

Você notará que `/challenge/run` ainda imprimirá mensagens no terminal, mesmo após você ter redirecionado o stdout.
Isso acontece porque o programa envia instruções e mensagens de feedback pela saída de erro padrão (`stderr`), imprimindo apenas a flag pela saída padrão (`stdout`)!

----
**AVISO:** Se algum teste anterior tiver deixado um arquivo, diretório ou symlink chamado `myflag` no seu diretório atual, remova-o antes para que a shell possa criar um arquivo gravável normal.
