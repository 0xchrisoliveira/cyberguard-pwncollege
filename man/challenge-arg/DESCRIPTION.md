A necessidade mais comum que você terá de consultar a documentação é simplesmente descobrir como usar a infinidade de programas disponíveis — e, em particular, descobrir quais argumentos passar na linha de comando.
Este módulo focará amplamente nesse conceito como base para entender o funcionamento geral de programas.
Ao longo do módulo, exploraremos diversas formas de solicitar ajuda ao sistema, mas primeiro vamos exercitar a leitura direta da documentação.

O uso correto dos programas depende, em grande parte, de fornecer os argumentos adequados.
Lembre-se do modificador `-a` em `ls -a` no desafio de arquivos ocultos do módulo [Compreendendo Comandos](/linux-luminarium/commands): aquele `-a` foi um _argumento_ que instruiu o `ls` a listar tanto arquivos ocultos quanto não ocultos.
Como o nosso objetivo _era_ listar arquivos ocultos, invocar `ls` com o argumento `-a` foi a escolha precisa para aquele cenário.

O programa deste desafio é `/challenge/challenge`, e você precisará invocá-lo corretamente para que ele entregue a flag.
Imagine que a documentação dele seja a seguinte:

> Bem-vindo à documentação do `/challenge/challenge`! Para executar este programa corretamente, você deve passar a ele o argumento `--giveflag`. Boa sorte!

Com base nessa informação, vá em frente e capture a flag!
