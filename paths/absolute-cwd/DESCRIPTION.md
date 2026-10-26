O sistema de arquivos do Linux possui toneladas de diretórios com milhares de arquivos.
Você pode navegar entre os diretórios usando o comando `cd` (*change directory*, mudar de diretório) e passando um caminho como argumento, desta forma:

```console
hacker@dojo:~$ cd /some/new/directory
hacker@dojo:/some/new/directory$
```

Isso altera o diretório atual de trabalho (*current working directory* ou `cwd`) do seu processo (neste caso, a shell bash).
Todo processo possui um diretório no qual está posicionado no momento.
Os motivos para isso ficarão claros mais adiante neste módulo.

Como curiosidade, agora você pode entender o que significava o `~` no prompt!
Ele exibe o caminho atual onde a sua shell está posicionada.

Este desafio exigirá que você execute o programa `/challenge/run` a partir de um diretório específico (que ele próprio informará).
Você precisará usar o comando `cd` para entrar nesse diretório antes de executar novamente o programa do desafio.
Boa sorte!
