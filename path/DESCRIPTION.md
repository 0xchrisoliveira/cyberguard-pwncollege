Até agora, você invocou comandos de várias maneiras:

- Por meio de um caminho absoluto (ex.: `/challenge/run`).
- Por meio de um caminho relativo (ex.: `./run`).
- Por meio apenas do nome do comando (ex.: `ls`).

Os dois primeiros casos — caminhos absolutos e relativos — são diretos: o arquivo `run` reside no diretório `/challenge` e ambos os casos apontam para ele (desde que, claro, o caminho relativo seja executado tendo `/challenge` como diretório de trabalho atual).
Mas e quanto ao último caso?
Onde o programa `ls` está localizado no sistema?
Como a shell sabe onde procurá-lo?

Neste módulo, vamos desvendar os bastidores e responder a essa pergunta!
Venha conosco.
