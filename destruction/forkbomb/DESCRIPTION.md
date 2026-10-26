Como você aprendeu no módulo [Processos e Tarefas](/linux-luminarium/processes), sempre que você inicia um programa, o sistema operacional Linux cria um novo processo.
Se você criar processos mais rápido do que o kernel consegue gerenciar, a tabela de processos ficará cheia e *absolutamente tudo* irá congelar.
Esse novo processo (por exemplo, a execução de um `ls`) é derivado ("bifurcado" ou *forked*) a partir de um processo pai (como a instância da sua shell).
Por conta disso, essa explosão induzida de processos é chamada de "Bomba Fork" (*Fork Bomb*).

Você já possui as ferramentas necessárias para fazer isso:

- Escrever um script simples (como no módulo [Encadeando Comandos](/linux-luminarium/chaining))
- Torná-lo executável (como no módulo [Compreendendo Permissões](/linux-luminarium/permissions))
- Fazer com que ele execute uma cópia de si mesmo em segundo plano (como no módulo [Processos e Tarefas](/linux-luminarium/processes))
- E então disparar _outra_ cópia de si mesmo em segundo plano!

Cada cópia iniciará mais duas, e cada uma dessas iniciará outras duas, inundando o sistema com tantos processos que novos programas não conseguirão ser inicializados!

Este desafio contém o utilitário `/challenge/check`, que verificará se a sua bomba fork está funcionando (ou seja, se o sistema não consegue mais criar novos processos) e entregará a flag caso o ataque tenha sucesso.
Certifique-se de executá-lo (em outro terminal) _antes_ de disparar o ataque; caso contrário, você não conseguirá abri-lo depois!

----
**NOTA:**
Escusado dizer que isso tornará o seu ambiente temporariamente inoperante.
Basta reiniciar o desafio (ou iniciar outro) para retornar tudo ao estado funcional normal!
