Agora já sabemos como listar, ler e criar arquivos.
Mas como fazemos para encontrá-los?
Usamos o comando `find`!

O comando `find` aceita argumentos opcionais que definem os critérios de busca e o local inicial da pesquisa.
Se você não especificar um critério de busca, o `find` corresponderá a todos os arquivos.
Se você não especificar um local de busca, o `find` utilizará o diretório atual de trabalho (`.`).
Por exemplo:

```console
hacker@dojo:~$ mkdir my_directory
hacker@dojo:~$ mkdir my_directory/my_subdirectory
hacker@dojo:~$ touch my_directory/my_file
hacker@dojo:~$ touch my_directory/my_subdirectory/my_subfile
hacker@dojo:~$ find
.
./my_directory
./my_directory/my_subdirectory
./my_directory/my_subdirectory/my_subfile
./my_directory/my_file
hacker@dojo:~$
```

E ao especificar um local de busca:

```console
hacker@dojo:~$ find my_directory/my_subdirectory
my_directory/my_subdirectory
my_directory/my_subdirectory/my_subfile
hacker@dojo:~$
```

E, claro, podemos especificar critérios!
Por exemplo, aqui filtramos por nome com a opção `-name`:

```console
hacker@dojo:~$ find -name my_subfile
./my_directory/my_subdirectory/my_subfile
hacker@dojo:~$ find -name my_subdirectory
./my_directory/my_subdirectory
hacker@dojo:~$
```

Você pode pesquisar em todo o sistema de arquivos se desejar:

```console
hacker@dojo:~$ find / -name hacker
/home/hacker
hacker@dojo:~$
```

Agora é a sua vez.
A flag foi escondida em um diretório arbitrário no sistema de arquivos.
O arquivo continua se chamando `flag`.
Vá e encontre-o!

Algumas observações importantes:
Primeiro, existem outros arquivos chamados `flag` espalhados pelo sistema de arquivos. Não se preocupe se o primeiro que você abrir não contiver a flag verdadeira.
Segundo, existem muitos locais no sistema de arquivos aos quais um usuário comum não tem permissão de acesso. Isso fará com que o `find` exiba mensagens de erro de permissão negada (*Permission denied*), mas você pode ignorá-las; a flag não está escondida nesses lugares restritos!
Por fim, uma busca completa com o `find` pode levar alguns instantes; seja paciente!
