Assim como a saída padrão, você também pode redirecionar o canal de erros dos comandos.
Aqui, aprenderemos sobre os *Descritores de Arquivo* (*File Descriptors* ou FDs).
Um Descritor de Arquivo é um número inteiro que identifica um canal de comunicação no Linux.
Você já os tem usado, mesmo sem perceber.
Já conhecemos os três principais:

- FD 0: Entrada Padrão (*Standard Input* - `stdin`)
- FD 1: Saída Padrão (*Standard Output* - `stdout`)
- FD 2: Saída de Erro Padrão (*Standard Error* - `stderr`)

Ao redirecionar a comunicação de um processo, você faz isso indicando o número do FD, embora alguns sejam implícitos.
Por exemplo, o operador `>` sem número implícita `1>`, redirecionando o FD 1 (`stdout`).
Portanto, os dois comandos a seguir são equivalentes:

```console
hacker@dojo:~$ echo hi > asdf
hacker@dojo:~$ echo hi 1> asdf
```

Redirecionar erros torna-se muito simples a partir deste conceito.
Se houver um comando que produza dados via erro padrão (como `/challenge/run`), você pode fazer:

```console
hacker@dojo:~$ /challenge/run 2> errors.log
```

Isso redirecionará o canal de erro padrão (FD 2) para o arquivo `errors.log`.
Além disso, você pode redirecionar múltiplos descritores de arquivo simultaneamente em um mesmo comando!
Por exemplo:

```console
hacker@dojo:~$ some_command > output.log 2> errors.log
```

Esse comando redirecionará a saída padrão para `output.log` e os erros para `errors.log`.

Vamos colocar isso em prática!
Neste desafio, você precisará redirecionar a saída de `/challenge/run` para `myflag`, e os "erros" (no nosso caso, as instruções e orientações) para `instructions`.
Você perceberá que nada aparecerá no terminal, pois você terá redirecionado absolutamente tudo!
Ao concluir a operação com sucesso, você encontrará as instruções em `instructions` e a sua flag em `myflag`!

----
**AVISO:** Se algum teste anterior tiver deixado um arquivo, diretório ou symlink chamado `myflag` ou `instructions` no seu diretório atual, remova-o antes para que a shell possa criar arquivos graváveis normais.
