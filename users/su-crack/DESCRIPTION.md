Quando você insere uma senha para o comando `su`, ele a compara com a senha armazenada para aquele usuário.
Essas senhas *costumavam* ser armazenadas no `/etc/passwd`, mas como o `/etc/passwd` precisa ser um arquivo legível por todos os usuários do sistema, isso não era seguro para senhas; por isso, elas foram migradas para o `/etc/shadow`.
Aqui está o exemplo de `/etc/shadow` do nível anterior:

```console
root:$6$s74oZg/4.RnUvwo2$hRmCHZ9rxX56BbjnXcxa0MdOsW2moiW8qcAl/Aoc7NEuXl2DmJXPi3gLp7hmyloQvRhjXJ.wjqJ7PprVKLDtg/:19921:0:99999:7:::
daemon:*:19873:0:99999:7:::
bin:*:19873:0:99999:7:::
sys:*:19873:0:99999:7:::
sync:*:19873:0:99999:7:::
games:*:19873:0:99999:7:::
man:*:19873:0:99999:7:::
lp:*:19873:0:99999:7:::
mail:*:19873:0:99999:7:::
news:*:19873:0:99999:7:::
uucp:*:19873:0:99999:7:::
proxy:*:19873:0:99999:7:::
www-data:*:19873:0:99999:7:::
backup:*:19873:0:99999:7:::
list:*:19873:0:99999:7:::
irc:*:19873:0:99999:7:::
gnats:*:19873:0:99999:7:::
nobody:*:19873:0:99999:7:::
_apt:*:19873:0:99999:7:::
systemd-timesync:*:19901:0:99999:7:::
systemd-network:*:19901:0:99999:7:::
systemd-resolve:*:19901:0:99999:7:::
mysql:!:19901:0:99999:7:::
messagebus:*:19901:0:99999:7:::
sshd:*:19901:0:99999:7:::
hacker::19916:0:99999:7:::
zardus:$6$bEFkpM0w/6J0n979$47ksu/JE5QK6hSeB7mmuvJyY05wVypMhMMnEPTIddNUb5R9KXgNTYRTm75VOu1oRLGLbAql3ylkVa5ExuPov1.:19921:0:99999:7:::
```

Separados por `:`, o primeiro campo de cada linha é o nome de usuário e o segundo é a senha criptografada em hash.
Um valor de `*` ou `!` significa funcionalmente que o login por senha para aquela conta está desativado; um campo vazio significa que não há senha (uma falha de configuração não muito rara que permite usar `su` sem senha em certos cenários); e uma cadeia longa como a de Zardus (`$6$bEFkpM0w/6J0n979$47ksu/JE5QK6hSeB7mmuvJyY05wVypMhMMnEPTIddNUb5R9KXgNTYRTm75VOu1oRLGLbAql3ylkVa5ExuPov1.`) é o resultado do hash unidirecional (*one-way hash*) da senha de Zardus do nível anterior (que era `dont-hack-me`).
Os demais campos desse arquivo têm outros significados e você pode ler mais sobre eles [aqui](https://www.cyberciti.biz/faq/understanding-etcshadow-file/).

Quando você digita uma senha no `su`, o utilitário gera o hash unidirecional dela e compara o resultado com o hash armazenado.
Se os hashes coincidirem, o `su` concede o acesso à conta!

Mas e se você não souber a senha?
Se você tiver o hash da senha, você pode *quebrá-lo* (*crack*)!
Embora o arquivo `/etc/shadow` seja, por padrão, legível apenas pelo `root`, vazamentos acontecem!
Por exemplo, backups frequentemente são armazenados desprotegidos ou sem criptografia em servidores de arquivos, o que já gerou inúmeros incidentes de vazamento de dados no mundo real.

Se um invasor obtiver uma cópia vazada do `/etc/shadow`, ele pode iniciar o processo de quebra de senhas (*cracking*).
A quebra pode ser realizada com a famosa ferramenta [John the Ripper](https://www.openwall.com/john/), desta forma:

```console
hacker@dojo:~$ john ./my-leaked-shadow-file
Loaded 1 password hash (crypt, generic crypt(3) [?/64])
Will run 32 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
password1337      (zardus)
1g 0:00:00:22 3/3 0.04528g/s 10509p/s 10509c/s 10509C/s lykys..lank
Use the "--show" option to display all of the cracked passwords reliably
Session completed
hacker@dojo:~$
```

Aqui, o John the Ripper quebrou o hash da senha vazada de Zardus e descobriu o valor real em texto claro: `password1337`.
Pobre Zardus!

Este nível simula exatamente esse cenário, fornecendo a você um vazamento do `/etc/shadow` (em `/challenge/shadow-leak`).
Quebre a senha (isso pode levar alguns minutos), faça `su` para o usuário `zardus` e execute `/challenge/run` para capturar a flag!
