Agora que já estamos bem familiarizados com a propriedade de arquivos, vamos falar sobre o outro lado da moeda: as permissões de arquivos (*file permissions*).
Relembre o nosso exemplo:

```console
hacker@dojo:~$ mkdir pwn_directory
hacker@dojo:~$ touch college_file
hacker@dojo:~$ ls -l
total 4
-rw-r--r-- 1 hacker hacker    0 May 22 13:42 college_file
drwxr-xr-x 2 hacker hacker 4096 May 22 13:42 pwn_directory
hacker@dojo:~$
```

Para recapitular: o primeiro caractere indica o tipo de arquivo.
Os nove caracteres seguintes são as permissões reais de acesso ao arquivo ou diretório, divididas em 3 caracteres indicando as permissões do usuário dono (agora você entende isso perfeitamente!), 3 caracteres indicando as permissões do grupo dono (o que você também já compreende!) e 3 caracteres indicando as permissões de qualquer outro usuário (*other* — todos os demais usuários e grupos do sistema).

Cada caractere do trio representa um tipo específico de permissão:

```
r - usuário/grupo/outros podem ler o arquivo (ou listar o conteúdo do diretório)
w - usuário/grupo/outros podem modificar o arquivo (ou criar/excluir arquivos no diretório)
x - usuário/grupo/outros podem executar o arquivo como programa (ou acessar o diretório, ex: usando `cd`)
- - nenhuma permissão
```

Para o arquivo `college_file` acima, a entrada de permissões `rw-r--r--` é decodificada assim:

- `r`: o usuário dono do arquivo (usuário `hacker`) pode lê-lo
- `w`: o usuário dono do arquivo (usuário `hacker`) pode escrever nele
- `-`: o usuário dono do arquivo (usuário `hacker`) *não* pode executá-lo
- `r`: usuários membros do grupo dono do arquivo (grupo `hacker`) podem lê-lo
- `-`: usuários membros do grupo dono do arquivo (grupo `hacker`) *não* podem escrever nele
- `-`: usuários membros do grupo dono do arquivo (grupo `hacker`) *não* podem executá-lo
- `r`: todos os outros usuários podem lê-lo
- `-`: todos os outros usuários *não* podem escrever nele
- `-`: todos os outros usuários *não* podem executá-lo

Agora, vejamos as permissões padrão do `/flag`:

```console
hacker@dojo:~$ ls -l /flag
-r-------- 1 root root 53 Jul  4 04:47 /flag
hacker@dojo:~$
```

Aqui, há apenas um bit ativo: a permissão de leitura (`r`) para o usuário proprietário (neste caso, `root`).
Os membros do grupo dono (o grupo `root`) e todos os outros usuários não possuem qualquer permissão de acesso.

Você pode estar se perguntando como funcionaram os níveis de `chgrp`, se não havia permissão para o grupo naquele arquivo.
Bem, naqueles níveis específicos, configurei as permissões de forma diferente:

```console
hacker@dojo:~$ ls -l /flag
-r--r----- 1 root root 53 Jul  4 04:47 /flag
hacker@dojo:~$
```

O grupo tinha permissão de leitura!
É por isso que mudar o grupo do arquivo via `chgrp` permitiu que você o lesse.

Enfim!
Assim como a propriedade, as permissões de arquivos também podem ser alteradas.
Isso é feito com o comando `chmod` (**ch**ange **mod**e).
A sintaxe básica do chmod é:

```
chmod [OPÇÕES] MODO ARQUIVO
```

Você pode especificar o `MODO` de duas formas: como uma modificação das permissões existentes ou definindo um modo totalmente novo que sobrescreve o anterior.

Neste nível, cobriremos a primeira forma: modificando o modo existente.
O `chmod` permite ajustar permissões no formato `QUEM`+/-`O_QUE`, onde `QUEM` representa o usuário (`u`), grupo (`g`), outros (`o`) ou todos (`a`), e `O_QUE` representa leitura (`r`), escrita (`w`) ou execução (`x`).
Por exemplo, para conceder permissão de *leitura* para o *usuário* dono, especificamos o modo `u+r`.
Permissões de escrita (`w`) e execução (`x`) para o grupo (`g`) e outros (`o`) — ou todos (`a`) — são definidas da mesma maneira.
Mais exemplos:

- `u+r`, como visto acima, adiciona permissão de leitura às permissões do usuário
- `g+wx` adiciona permissões de escrita e execução ao grupo
- `o-w` *remove* a permissão de escrita de outros usuários
- `a-rwx` remove todas as permissões do usuário, do grupo e de outros

Veja na prática:

```console
root@dojo:~# mkdir pwn_directory
root@dojo:~# touch college_file
root@dojo:~# ls -l
total 4
-rw-r--r-- 1 root root    0 May 22 13:42 college_file
drwxr-xr-x 2 root root 4096 May 22 13:42 pwn_directory
root@dojo:~# chmod go-rwx *
root@dojo:~# ls -l
total 4
-rw------- 1 hacker root    0 May 22 13:42 college_file
drwx------ 2 root   root 4096 May 22 13:42 pwn_directory
root@dojo:~#
```

Neste desafio, você deve alterar as permissões do arquivo `/flag` para conseguir lê-lo!
Normalmente, você precisaria ser o dono do arquivo para alterar suas permissões, mas tornei o comando `chmod` todo-poderoso neste nível: você pode executar `chmod` em qualquer arquivo que quiser, mesmo sendo o usuário `hacker`.
Esse é um poder absoluto!
O arquivo `/flag` pertence ao `root`, e você não pode mudar o dono dele aqui, mas pode torná-lo legível para o seu usuário.
Vá em frente e resolva o desafio!
