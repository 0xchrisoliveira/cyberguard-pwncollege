Como você viu no módulo anterior, existem muitos casos em que usuários comuns (não-`root`) precisam de privilégios elevados para executar certas tarefas do sistema.
O administrador do sistema não pode estar sempre presente para digitar a senha de root toda vez que um usuário precisar executar uma tarefa que apenas o `root` ou membros do `sudoers` têm permissão para fazer.
Em vez disso, o bit de permissão especial "Set User ID" (SUID) permite que qualquer usuário execute um programa com os privilégios do usuário proprietário daquele arquivo executável.

Esse é exatamente o mecanismo utilizado pelos programas dos desafios para conseguir ler a flag ou, no mundo Linux fora do pwn.college, para viabilizar ferramentas administrativas essenciais como `su`, `sudo` e outras.
As permissões de um arquivo com o bit SUID se parecem com isto:

```console
hacker@dojo:~$ ls -l /usr/bin/sudo
-rwsr-xr-x 1 root root 232416 Dec 1 11:45 /usr/bin/sudo
hacker@dojo:~$
```

A letra `s` no lugar do bit tradicional de execução (`x`) indica que o programa é executável *com SUID*.
Isso significa que, independentemente de qual usuário execute o programa (desde que possua permissão de execução), o programa rodará com os privilégios do usuário proprietário (neste caso, o usuário `root`).

Como proprietário de um arquivo, você pode ativar o bit SUID usando o comando chmod:

```
chmod u+s [program]
```

Mas muito cuidado!
Conceder o bit SUID a um executável cujo dono é o `root` pode fornecer a invasores um vetor direto de escalação de privilégios para se tornarem `root`.
Você aprenderá muito mais sobre isso no [módulo Program Misuse](/fundamentals/program-misuse/).

Agora, nós permitiremos que você adicione o bit SUID ao programa `/challenge/getroot` para abrir uma shell de `root` e você mesmo conseguir fazer `cat` na flag!
