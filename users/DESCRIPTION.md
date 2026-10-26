Você achou que você, o `hacker`, era o único habitante nesta máquina?
Existem MUITOS usuários em um sistema Linux típico!
A lista completa de usuários de um sistema Linux é especificada no arquivo `/etc/passwd` (chamado assim por razões históricas — ele não armazena mais as senhas de fato).
Aqui está um exemplo extraído do container do dojo:

```console
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:101:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
mysql:x:104:105:MySQL Server,,,:/nonexistent:/bin/false
messagebus:x:105:106::/nonexistent:/usr/sbin/nologin
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
hacker:x:1000:1000::/home/hacker:/bin/bash
```

São muitos usuários e muitas informações!
Cada linha contém, separados por `:`, o nome de usuário (*username*), um `x` como marcador de posição para onde a senha *costumava* ficar (veremos onde ela fica de verdade mais adiante), o ID numérico do usuário (UID), o ID numérico do grupo padrão (GID), detalhes descritivos do usuário, o diretório pessoal (*home directory*) e a shell padrão.

Podemos ver o usuário `hacker` lá embaixo.
Esse é você!
A maior parte dos outros usuários existe por motivos históricos, são contas de serviço para suportar softwares instalados ou são contas de "utilidade" (por exemplo, o usuário `nobody` é utilizado para garantir que certos programas rodem sem nenhum privilégio especial).

Um usuário fundamental é o `root`: o administrador do sistema.
O administrador do sistema possui implicações óbvias de segurança: um usuário comum como `hacker` que consiga, através de diversas funcionalidades do Linux, se tornar o usuário `root`, terá controle total sobre o sistema.
Um objetivo muito frequente de invasores é justamente escalar privilégios para `root`, e por isso a conta `root` deve ser protegida a todo custo!

Neste módulo, exploraremos diversas peculiaridades sobre contas de usuários, aprenderemos os métodos formais de alternar usuários para administrar o sistema e nos divertiremos pelo caminho!
