Por mais tentador que seja, usar `*` para encurtar o que precisa ser digitado na linha de comando pode levar a erros graves.
Seu glob pode se expandir para arquivos indesejados, e você pode só perceber quando o comando `rm` já estiver em execução!
Ninguém está imune a esse tipo de falha.

Uma alternativa mais segura quando você deseja especificar um alvo concreto é o recurso de *autocompletar com Tab* (*tab completion*).
Se você pressionar a tecla Tab na shell, ela tentará deduzir o que você está prestes a digitar e completará automaticamente.
O autocompletar é incrivelmente útil, e este desafio explorará seu uso ao especificar arquivos.

Neste desafio, copiamos a flag para `/challenge/pwncollege`, e você pode ler esse arquivo livremente com `cat`.
Porém, você não conseguirá digitar o nome do arquivo diretamente: usamos um truque astuto para garantir que você *obrigatoriamente* use o autocompletar com Tab.
Experimente:

```console
hacker@dojo:~$ ls /challenge
Dockerfile  pwncollege
hacker@dojo:~$ cat /challenge/pwncollege
cat: /challenge/pwncollege: No such file or directory
hacker@dojo:~$ cat /challenge/pwn<TAB>
pwn.college{HECK YEAH}
hacker@dojo:~$
```

Ao pressionar a tecla Tab, o nome do arquivo se expandirá e você conseguirá ler seu conteúdo.
Boa sorte!
