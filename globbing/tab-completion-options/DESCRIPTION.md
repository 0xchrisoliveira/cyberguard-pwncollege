Considere a seguinte situação:

```console
hacker@dojo:~$ ls
flag  flamingo  flowers
hacker@dojo:~$ cat f<TAB>
```

Existem múltiplas opções possíveis!
O que acontece?

O comportamento varia conforme a shell específica e suas configurações.
Por padrão, o `bash` completará automaticamente até o ponto em que houver ambiguidade (neste caso, `fl`).
Quando você pressionar Tab uma *segunda* vez, ele exibirá todas as opções correspondentes.
Outras shells e configurações podem, em vez disso, alternar ciclicamente entre as opções.

Este desafio possui um diretório `/challenge/files` com uma série de arquivos começando com `pwncollege`.
Use o autocompletar com Tab a partir de `/challenge/files/p` ou similar, e encontre o caminho até a flag!
