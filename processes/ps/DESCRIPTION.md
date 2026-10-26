Primeiro, aprenderemos a listar os processos em execução utilizando o comando `ps`.
Dependendo de a quem você perguntar, `ps` significa "process snapshot" (captura de processos) ou "process status" (estado dos processos), e serve para listar processos.
Por padrão, o `ps` lista apenas os processos em execução no seu terminal atual, o que, francamente, não é muito útil:

```console
hacker@dojo:~$ ps
    PID TTY          TIME CMD
    329 pts/0    00:00:00 bash
    349 pts/0    00:00:00 ps
hacker@dojo:~$
```

No exemplo acima, temos a própria shell (`bash`) e o próprio processo `ps`, sendo apenas isso o que está rodando naquele terminal específico.
Também vemos que cada processo possui um identificador numérico (o *Process ID*, ou PID), que é um número único para identificar cada processo em execução em um ambiente Linux.
Vemos também o terminal no qual os comandos estão rodando (neste caso, a designação `pts/0`) e a quantidade total de *tempo de CPU* que o processo consumiu até o momento (como esses processos são muito leves, não consumiram sequer 1 segundo!).

Na maioria dos casos, isso é tudo o que você verá com um `ps` padrão.
Para torná-lo realmente útil, precisamos passar alguns argumentos.

Como o `ps` é um utilitário bastante antigo, o uso de seus argumentos tem algumas peculiaridades históricas.
Existem duas formas principais de especificar argumentos:

**Sintaxe "Padrão" (Standard/POSIX):** nesta sintaxe, você pode usar `-e` para listar "todos" (*every*) os processos e `-f` para uma saída em "formato completo" (*full format*), incluindo os argumentos.
Eles podem ser combinados em um único argumento: `-ef`.

**Sintaxe "BSD":** nesta sintaxe, você usa `a` para listar processos de todos os usuários (*all*), `x` para listar processos que não estão vinculados a um terminal e `u` para uma saída legível voltada ao usuário (*user-readable*).
Eles podem ser combinados em um único argumento: `aux`.

Esses dois métodos, `ps -ef` e `ps aux`, produzem saídas ligeiramente diferentes, mas bastante equivalentes e facilmente reconhecíveis.

Vamos testar no dojo:

```console
hacker@dojo:~$ ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
hacker         1       0  0 05:34 ?        00:00:00 /sbin/docker-init -- /bin/sleep 6h
hacker         7       1  0 05:34 ?        00:00:00 /bin/sleep 6h
hacker       102       1  1 05:34 ?        00:00:00 /usr/lib/code-server/lib/node /usr/lib/code-server --auth=none -
hacker       138     102 11 05:34 ?        00:00:07 /usr/lib/code-server/lib/node /usr/lib/code-server/out/node/entr
hacker       287     138  0 05:34 ?        00:00:00 /usr/lib/code-server/lib/node /usr/lib/code-server/lib/vscode/ou
hacker       318     138  6 05:34 ?        00:00:03 /usr/lib/code-server/lib/node --dns-result-order=ipv4first /usr/
hacker       554     138  3 05:35 ?        00:00:00 /usr/lib/code-server/lib/node /usr/lib/code-server/lib/vscode/ou
hacker       571     554  0 05:35 pts/0    00:00:00 /usr/bin/bash --init-file /usr/lib/code-server/lib/vscode/out/vs
hacker       695     571  0 05:35 pts/0    00:00:00 ps -ef
hacker@dojo:~$
```

Você pode ver aqui que existem processos rodando para a inicialização do ambiente do desafio (`docker-init`), um temporizador para encerrar o ambiente e preservar recursos computacionais (`sleep 6h` para encerrar após 6 horas), o ambiente do VSCode (vários processos auxiliares do `code-server`), a shell (`bash`) e o próprio comando `ps -ef`.
É essencialmente o mesmo resultado com `ps aux`:

```
hacker@dojo:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
hacker         1  0.0  0.0   1128     4 ?        Ss   05:34   0:00 /sbin/docker-init -- /bin/sleep 6h
hacker         7  0.0  0.0   2736   580 ?        S    05:34   0:00 /bin/sleep 6h
hacker       102  0.4  0.0 723944 64660 ?        Sl   05:34   0:00 /usr/lib/code-server/lib/node /usr/lib/code-serve
hacker       138  3.3  0.0 968792 106272 ?       Sl   05:34   0:07 /usr/lib/code-server/lib/node /usr/lib/code-serve
hacker       287  0.0  0.0 717648 53136 ?        Sl   05:34   0:00 /usr/lib/code-server/lib/node /usr/lib/code-serve
hacker       318  3.3  0.0 977472 98256 ?        Sl   05:34   0:06 /usr/lib/code-server/lib/node --dns-result-order=
hacker       554  0.4  0.0 650560 55360 ?        Rl   05:35   0:00 /usr/lib/code-server/lib/node /usr/lib/code-serve
hacker       571  0.0  0.0   4600  4032 pts/0    Ss   05:35   0:00 /usr/bin/bash --init-file /usr/lib/code-server/li
hacker      1172  0.0  0.0   5892  2924 pts/0    R+   05:38   0:00 ps aux
hacker@dojo:~$
```

Há muitas semelhanças entre o `ps -ef` e o `ps aux`: ambos exibem o usuário (coluna `UID`/`USER`), o PID, o TTY, o horário de início do processo (`STIME`/`START`), o tempo total de CPU utilizado (`TIME`) e o comando executado (`CMD`/`COMMAND`).
O `ps -ef` exibe adicionalmente o *Parent Process ID* (`PPID`), que é o PID do processo pai que iniciou o processo em questão, enquanto o `ps aux` mostra o percentual de uso de CPU e de memória que o processo está utilizando no sistema.
Além disso, há diversos outros detalhes nos quais não precisamos nos aprofundar agora.

De qualquer forma, vamos praticar!
Neste nível, renomeei novamente `/challenge/run` para um nome de arquivo aleatório e, desta vez, fiz com que você não possa listar (`ls`) o diretório `/challenge`!
No entanto, eu também o coloquei em execução. Portanto, você pode encontrá-lo na lista de processos ativos, descobrir o nome do arquivo executável e executá-lo diretamente com seu caminho completo para obter a flag!
Boa sorte!

**NOTA:** Tanto o `ps -ef` quanto o `ps aux` truncam a exibição do comando de acordo com a largura do seu terminal (razão pela qual os exemplos acima ficam alinhados certinhos no canto direito da tela).
Se você não conseguir ler o caminho completo do processo, aumente a largura da janela do terminal (ou redirecione a saída para um arquivo ou pipe para evitar esse truncamento)!
Como alternativa, você pode passar a opção `w` *duas vezes* (por exemplo, `ps -efww` ou `ps auxww`) para desativar o truncamento de linha.
