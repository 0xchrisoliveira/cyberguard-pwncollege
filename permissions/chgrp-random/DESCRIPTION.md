Nos níveis anteriores, você deve ter notado que o usuário `hacker` pertence ao grupo `hacker` e que `zardus` pertence ao grupo `zardus`.
Existe uma convenção no Linux em que cada usuário tem seu próprio grupo homônimo, mas isso não é uma regra obrigatória.
Por exemplo, muitos laboratórios de informática colocam todos os seus usuários em um único grupo compartilhado chamado `users`.

O ponto aqui é: antes você usou o grupo `hacker`, mas neste nível isso não funcionará.
Ainda permitirei que você use `chgrp`, mas defini aleatoriamente o nome do grupo em que seu usuário está inserido.
Você precisará usar o comando `id` para descobrir qual é esse grupo e, em seguida, usar `chgrp` rumo à vitória!
