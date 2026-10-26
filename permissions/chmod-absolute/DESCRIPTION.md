Além de adicionar e remover permissões pontuais, como no nível anterior, o comando `chmod` também pode definir diretamente o conjunto completo de permissões, sobrescrevendo as anteriores.
Isso é feito utilizando o operador `=` em vez de `-` ou `+`.
Por exemplo:

- `u=rw` define permissões de leitura e escrita para o usuário dono e remove a permissão de execução
- `o=x` define apenas permissão de execução para outros (*world*), removendo leitura e escrita
- `a=rwx` define permissões de leitura, escrita e execução para todos: usuário dono, grupo e outros!

Mas e se você quiser alterar as permissões do usuário de uma forma e as do grupo de outra?
Digamos que você queira definir `rw` para o usuário dono, mas apenas `r` para o grupo dono?
Você pode fazer isso encadeando múltiplos modos no `chmod` separados por vírgula (`,`)!

- `chmod u=rw,g=r /challenge/pwn` definirá permissões de leitura e escrita para o usuário e somente leitura para o grupo
- `chmod a=r,u=rw /challenge/pwn` definirá leitura e escrita para o usuário e somente leitura para o grupo e outros

Além disso, você pode zerar completamente as permissões utilizando `-` após o `=`:

- `chmod u=rw,g=r,o=- /challenge/pwn` definirá permissões de leitura e escrita para o usuário, somente leitura para o grupo e absolutamente nenhuma permissão para outros

Tenha em mente que esse `-`, ao aparecer após `=`, está em um contexto diferente de quando aparece diretamente após `u`, `g` ou `o` (caso em que removeria bits específicos, e não zeraria o bloco).

Este nível aprofunda o nível anterior exigindo alterações de permissões mais radicais, nas quais você precisará utilizar `=` e encadeamento por vírgulas (`,`) para atingir o objetivo.
Boa sorte!
