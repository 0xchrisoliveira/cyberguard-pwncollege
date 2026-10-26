Acontece que você pode "cortar o intermediário" e eliminar a necessidade de salvar resultados em um arquivo em disco, como fizemos no nível anterior.
Você faz isso utilizando o operador `|` (*pipe* ou canalização).
A saída padrão (`stdout`) do comando à esquerda do pipe é conectada diretamente à entrada padrão (`stdin`) do comando à direita.
Por exemplo:

```
hacker@dojo:~$ echo no-no | grep yes
hacker@dojo:~$ echo yes-yes | grep yes
yes-yes
hacker@dojo:~$ echo yes-yes | grep no
hacker@dojo:~$ echo no-no | grep no
no-no
```

Agora experimente por conta própria! O programa `/challenge/run` gerará cem mil linhas de texto, incluindo a flag.
Use o `grep` via pipe para capturar a flag diretamente na saída em tempo real!
