Todo usuário possui um _diretório home_ (diretório pessoal), geralmente localizado sob `/home` no sistema de arquivos.
No dojo, você é o usuário `hacker`, e seu diretório home é `/home/hacker`.
O diretório home é tipicamente onde os usuários guardam a maior parte dos seus arquivos pessoais.
À medida que você avança pelo pwn.college, é aqui que você armazenará a maior parte dos seus scripts e soluções.

Tipicamente, sua sessão de shell começará com o seu diretório home como diretório de trabalho atual.
Considere o prompt inicial:

```console
hacker@dojo:~$
```

O `~` neste prompt representa o diretório de trabalho atual, sendo o `~` um atalho para `/home/hacker`.
O Bash fornece e utiliza esse atalho porque, novamente, a maior parte do seu tempo será gasta dentro do seu diretório pessoal.
Portanto, sempre que o bash encontra o `~` no início de um argumento de forma consistente com um caminho, ele o expande automaticamente para o seu diretório home.
Veja o exemplo:

```console
hacker@dojo:~$ echo LOOK: ~
LOOK: /home/hacker
hacker@dojo:~$ cd /
hacker@dojo:/$ cd ~
hacker@dojo:~$ cd ~/asdf
hacker@dojo:~/asdf$ cd ~/asdf
hacker@dojo:~/asdf$ cd ~
hacker@dojo:~$ cd /home/hacker/asdf
hacker@dojo:~/asdf$
```

Observe que a expansão do `~` resulta em um caminho _absoluto_, e apenas o `~` inicial é expandido.
Isso significa, por exemplo, que `~/~` será expandido para `/home/hacker/~`, e não para `/home/hacker/home/hacker`.

Curiosidade útil: o comando `cd` sem argumentos usará o seu diretório home como destino padrão:

```console
hacker@dojo:~$ cd /tmp
hacker@dojo:/tmp$ cd
hacker@dojo:~$
```

Agora é a sua vez de jogar!
Neste desafio, o programa `/challenge/run` gravará uma cópia da flag em qualquer arquivo que você especificar como argumento na linha de comando, atendendo às seguintes restrições:

1. O seu argumento deve ser um caminho absoluto.
2. O caminho deve estar dentro do seu diretório home.
3. Antes da expansão pelo shell, o seu argumento deve conter no máximo três caracteres.

Lembrando: você deve fornecer o seu caminho como um _argumento_ para `/challenge/run`, desta forma:

```console
hacker@dojo:~$ /challenge/run SEU_CAMINHO_AQUI
```
