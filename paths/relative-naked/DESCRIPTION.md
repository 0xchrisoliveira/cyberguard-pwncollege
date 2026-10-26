Agora você já está familiarizado com o conceito de referenciar caminhos absolutos e mudar de diretórios.
Se você utilizar caminhos absolutos para tudo, então realmente não importa em qual diretório você esteja, como você provavelmente percebeu nos três desafios anteriores.

No entanto, o diretório de trabalho atual importa — e muito — para caminhos **relativos**.

- Um caminho relativo é qualquer caminho que não começa na raiz (ou seja, que não começa com `/`).
- Um caminho relativo é interpretado de forma **relativa** ao seu **d**iretório **a**tual de **t**rabalho (*current working directory* ou `cwd`).
- O seu `cwd` é o diretório onde o seu prompt está localizado no momento.

Isso significa que a forma de especificar um determinado arquivo depende de onde o prompt do terminal está posicionado.

Imagine que queiramos acessar um arquivo localizado em `/tmp/a/b/my_file`.

- Se o meu `cwd` for `/`, um caminho relativo até o arquivo será `tmp/a/b/my_file`.
- Se o meu `cwd` for `/tmp`, um caminho relativo até o arquivo será `a/b/my_file`.
- Se o meu `cwd` for `/tmp/a/b/c`, um caminho relativo até o arquivo será `../my_file`. O `..` refere-se ao diretório pai (um nível acima).

Mudar de diretório e invocar um programa são duas ações separadas.
Por exemplo:

```console
hacker@dojo:~$ cd /
hacker@dojo:/$ bin/echo Hello
Hello
```

O primeiro comando muda o diretório de trabalho atual para `/`.
O segundo comando executa `/bin/echo` usando o caminho relativo `bin/echo`.

Vamos praticar aqui!
Primeiro, mude seu diretório de trabalho atual para `/`.
Em seguida, invoque `/challenge/run` usando um caminho relativo.
Para este nível, aqui vai uma dica:
O seu caminho relativo começa com a letra `c` 😊
