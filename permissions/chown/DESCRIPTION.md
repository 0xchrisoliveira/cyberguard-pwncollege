Primeiras coisas primeiro: propriedade de arquivos (*file ownership*).
Todo arquivo no Linux pertence a um usuário do sistema.
Na maior parte das vezes, no seu dia a dia, esse usuário é a conta com a qual você faz login na máquina.

Em um sistema compartilhado (como em um laboratório universitário), pode haver várias pessoas com diferentes contas de usuário, cada uma com seus próprios arquivos em seus respectivos diretórios pessoais (*home directories*).
Mas mesmo em um sistema não compartilhado (como seu computador pessoal), o Linux mantém várias contas de serviço para diferentes tarefas internas do sistema.

As duas contas de usuário mais importantes são:

1. A sua conta de usuário! No pwn.college, este é o usuário `hacker`, independentemente do seu nome real de usuário.
2. `root`. Esta é a conta administrativa e, na maioria dos cenários de segurança ofensiva, o grande prêmio. Se você assumir o controle do usuário `root`, você praticamente completou seu objetivo de invasão!

E o que isso tem a ver?
Bem, acontece que a maneira pela qual impedimos que você simplesmente faça `cat /flag` é definindo o `/flag` como pertencente ao usuário `root`, configurando suas permissões para que nenhum outro usuário possa lê-lo (você aprenderá a fazer isso mais adiante) e configurando o binário do desafio para rodar como `root` (o que também aprenderemos depois).
O resultado é que, quando você tenta fazer `cat /flag`, você recebe:

```console
hacker@dojo:~$ ls -l /flag
-r-------- 1 root root 53 Jul  4 04:47 /flag
hacker@dojo:~$ cat /flag
cat: /flag: Permission denied
hacker@dojo:~$
```

Aqui, você pode ver que a flag pertence ao usuário `root` (o primeiro `root` da linha) e ao grupo `root` (o segundo `root` da linha).
Quando tentamos lê-la como usuário `hacker`, nosso acesso é negado.
No entanto, se fôssemos o `root` (o sonho de todo hacker!), não teríamos qualquer problema para ler o arquivo:

```console
root@dojo:~# cat /flag
pwn.college{demo_flag}
root@dojo:~#
```

Curiosamente, nós podemos mudar a propriedade de arquivos!
Isso é feito através do comando `chown` (**ch**ange **own**er):

```
chown [username] [file]
```

Normalmente, o `chown` só pode ser executado pelo usuário `root`.
Vamos fingir que somos o `root` novamente (isso nunca perde a graça!) e observar um uso típico do `chown`:

```console
root@dojo:~# mkdir pwn_directory
root@dojo:~# touch college_file
root@dojo:~# ls -l
total 4
-rw-r--r-- 1 root root    0 May 22 13:42 college_file
drwxr-xr-x 2 root root 4096 May 22 13:42 pwn_directory
root@dojo:~# chown hacker college_file
root@dojo:~# ls -l
total 4
-rw-r--r-- 1 hacker root    0 May 22 13:42 college_file
drwxr-xr-x 2 root   root 4096 May 22 13:42 pwn_directory
root@dojo:~#
```

O dono de `college_file` foi alterado para o usuário `hacker`, e agora o `hacker` pode fazer com ele tudo o que o `root` conseguia fazer!
Se esse fosse o arquivo `/flag`, significaria que o usuário `hacker` agora poderia lê-lo!

Neste nível, praticaremos a alteração do dono do arquivo `/flag` para o usuário `hacker` e, em seguida, leremos a flag.
Exclusivamente para este desafio, configurei o ambiente para que você possa usar o comando `chown` à vontade mesmo como usuário `hacker` (lembrando que, normalmente, isso exige privilégios de `root`).
Use esse poder com sabedoria e mãos à obra com o chown!
