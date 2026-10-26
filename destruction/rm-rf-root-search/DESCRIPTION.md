Então você consegue sobreviver sem o `cat`!
Mas e sem o `ls`?
Desta vez, o `/challenge/check` restaurará a flag em um arquivo com _nome gerado aleatoriamente_.
Você precisará encontrá-lo sem poder recorrer ao utilitário `ls`.

Existem várias formas de solucionar este desafio.
O comando `echo` é embutido na shell (*builtin*), e você pode usar [Globbing de Arquivos](/linux-luminarium/globbing) como argumento para ele expandir todos os arquivos!
Por exemplo, `echo *` listará os nomes de todos os arquivos no diretório atual.
Da mesma forma, você pode utilizar o preenchimento automático com a tecla Tab (pressione Tab algumas vezes) para fazer a própria shell listar os arquivos disponíveis para você.

Qualquer que seja o caminho escolhido, localize o arquivo de nome aleatório criado pelo `/challenge/check` na raiz `/` após a execução do `rm`, leia o seu conteúdo e conquiste a flag!
