Este módulo apresentará a você o sistema de permissões do Linux, que é uma das partes mais fundamentais da sua jornada daqui em diante e controla o acesso a arquivos entre diferentes usuários.

No Linux, os arquivos possuem diferentes permissões ou modos de arquivo (*file modes*).
Você pode conferir as permissões de um arquivo ou diretório usando `ls -l`.
Vamos criar alguns arquivos e examinar suas permissões:

```console
hacker@dojo:~$ mkdir pwn_directory
hacker@dojo:~$ touch college_file
hacker@dojo:~$ ls -l
total 4
-rw-r--r-- 1 hacker hacker    0 May 22 13:42 college_file
drwxr-xr-x 2 hacker hacker 4096 May 22 13:42 pwn_directory
hacker@dojo:~$
```

Há muitas informações aí, e aprenderemos sobre boa parte delas neste módulo!
Por enquanto, vamos analisar a saída acima em linhas gerais:

#### O Tipo de Arquivo

O primeiro caractere de cada linha representa o tipo de arquivo.
No caso de `pwn_directory`, o `d` indica que se trata de um diretório (*directory*), e no caso de `college_file`, o traço `-` indica que é um arquivo comum.
Existem outros tipos também, e você encontrará alguns deles mais adiante em sua jornada no pwn.college.

#### As Permissões

Os nove caracteres seguintes representam as permissões de acesso propriamente ditas do arquivo ou diretório. Eles são divididos em 3 blocos de 3 caracteres: os 3 primeiros indicam as permissões que o usuário dono do arquivo (chamado de "dono" ou *owner*) possui; os 3 seguintes indicam as permissões do grupo dono do arquivo (*group*); e os últimos 3 caracteres indicam as permissões de qualquer outro usuário (*others* — outros usuários e outros grupos).
Aprenderemos detalhadamente sobre cada uma delas ao longo deste módulo.

#### Informações de Propriedade (Ownership)

Existem duas colunas que mostram o *usuário* dono do arquivo (neste caso, o usuário `hacker`) e, em seguida, o *grupo* dono do arquivo (neste caso, também o grupo `hacker`).
Você trabalhará bastante com isso por aqui!

----

Neste módulo, você praticará a percepção e manipulação de permissões.
Vamos começar!
