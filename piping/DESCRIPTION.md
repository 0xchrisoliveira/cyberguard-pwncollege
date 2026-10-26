Você deve ter percebido que alguns comandos exibem dados no seu terminal quando executados.
Até agora, isso serviu para imprimir diversas flags na sua tela, mas, como tantas coisas no Linux, a engenharia por trás disso vai muito mais fundo.
Os mecanismos de manipulação de entrada e saída na linha de comando são os grandes responsáveis pelo verdadeiro poder do terminal.

Este módulo ensinará você sobre *redirecionamento de entrada e saída* (*I/O redirection*).
Em termos simples, todo processo no Linux possui três canais padrão e iniciais de comunicação:

- **Entrada Padrão** (*Standard Input*): canal pelo qual o processo recebe dados de entrada. Por exemplo, a sua shell utiliza a Entrada Padrão para ler os comandos digitados por você.
- **Saída Padrão** (*Standard Output*): canal pelo qual os processos exibem dados normais, como as flags exibidas nos desafios anteriores ou a lista de arquivos retornada por comandos como `ls`.
- **Saída de Erro Padrão** (*Standard Error*): canal pelo qual os processos enviam detalhes de erros e mensagens de diagnóstico. Por exemplo, se você digitar incorretamente um comando, a shell avisará pela saída de erro que o comando não existe.

Por serem utilizados a todo momento no Linux, esses três canais são conhecidos por suas siglas abreviadas: `stdin`, `stdout` e `stderr`.
Neste módulo, você aprenderá a redirecionar, encadear via pipes, duplicar e manipular esses fluxos como um verdadeiro especialista.
Boa sorte!
