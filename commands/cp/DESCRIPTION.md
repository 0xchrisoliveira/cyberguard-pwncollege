Mas e se você quiser manter o arquivo original? Você pode fazer isso com o comando `cp` (*copy*).
O modo de uso é idêntico ao do `mv`, com a diferença de que ele preserva o arquivo de origem intacto.
O comando segue a sintaxe `cp ORIGEM DESTINO`: o primeiro argumento é o arquivo existente, e o segundo é o local onde a cópia deve ser criada.

Este desafio quer que você copie o arquivo `/flag` para `/tmp/hack-the-planet` (mãos à obra)!
Quando terminar, execute `/challenge/check`, que verificará a cópia e entregará a flag para você.

----
**NOTA:**
Quando o destino de um `cp` é um diretório, o `cp` coloca a cópia dentro dele preservando o nome do arquivo original.
Para este desafio, `/tmp/hack-the-planet` deve ser o nome do próprio arquivo copiado, e não um diretório existente.
