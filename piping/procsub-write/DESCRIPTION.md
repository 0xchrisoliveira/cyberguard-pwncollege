Você aprendeu que a substituição de processos permite tratar a saída de um comando como se fosse um arquivo de leitura com `<(comando)`.
Mas você também pode usar a substituição de processos para *escrever* dados na entrada de comandos!

Vimos que podemos duplicar dados para dois arquivos com `tee`:

```console
hacker@dojo:~$ echo HACK | tee THE > PLANET
hacker@dojo:~$ cat THE
HACK
hacker@dojo:~$ cat PLANET
HACK
hacker@dojo:~$
```

E também vimos o `tee` duplicar dados para um arquivo e para um comando simultaneamente:

```console
hacker@dojo:~$ echo HACK | tee THE | cat
HACK
hacker@dojo:~$ cat THE
HACK
hacker@dojo:~$
```

Mas e se quisermos duplicar o fluxo de dados para **dois comandos diferentes**?
Como a manpage do `tee` explica, ele foi concebido para gravar em arquivos e na saída padrão:

```text
TEE(1)                           User Commands                          TEE(1)

NAME
       tee - read from standard input and write to standard output and files
```

Mas espere! Você acabou de aprender que o bash pode fazer comandos se comportarem como arquivos usando a substituição de processos!
Para escrever na entrada de um comando (substituição de processo para saída), utilizamos `>(comando)`.
Se você passar o argumento `>(rev)`, o bash inicializará o comando `rev` (que lê dados do stdin, inverte a ordem dos caracteres e imprime no stdout), mas conectará a entrada dele a um arquivo de pipe nomeado temporário.
Quando outros comandos escrevem nesse arquivo, os dados são enviados diretamente para a entrada padrão do comando:

```console
hacker@dojo:~$ echo HACK | rev
KCAH
hacker@dojo:~$ echo HACK | tee >(rev)
HACK
KCAH
```

No exemplo acima, a seguinte sequência de eventos ocorreu:

1. O `bash` iniciou o comando `rev`, conectando um pipe nomeado (como `/dev/fd/63`) à entrada padrão do `rev`.
2. O `bash` iniciou o comando `tee`, conectando um pipe ao seu stdin e substituindo o primeiro argumento de `tee` por `/dev/fd/63`. O `tee` sequer tomou conhecimento de `>(rev)`; a shell realizou a *substituição* antes de disparar o `tee`.
3. O `bash` usou o comando embutido `echo` para enviar `HACK` ao stdin do `tee`.
4. O `tee` leu `HACK`, escreveu na saída padrão e em seguida gravou em `/dev/fd/63` (que está conectado ao stdin do `rev`).
5. O `rev` leu `HACK` do seu stdin, inverteu os caracteres e escreveu `KCAH` na saída padrão.

Agora é a sua vez!
Neste desafio, temos `/challenge/hack`, `/challenge/the` e `/challenge/planet`.
Execute o comando `/challenge/hack` e duplique a sua saída como entrada para ambos os comandos `/challenge/the` e `/challenge/planet`!
Lembre-se de que o `tee` aceita múltiplos alvos de saída, e `>(comando)` é uma forma válida de alvo.
Revise os desafios anteriores sobre `tee` e substituição de processos caso queira relembrar os detalhes.

----
**Curiosidade:**

O leitor atento perceberá que as seguintes linhas produzem o mesmo efeito:

```console
hacker@dojo:~$ echo hi | rev
ih
hacker@dojo:~$ echo hi > >(rev)
ih
hacker@dojo:~$
```

Existem muitas formas de encadear fluxos de dados!
Naturalmente, a segunda alternativa é muito menos legível e mais difícil de manter.
Por exemplo:

```console
hacker@dojo:~$ echo hi | rev | rev
hi
hacker@dojo:~$ echo hi > >(rev | rev)
hi
hacker@dojo:~$
```

Isso fica desnecessariamente confuso!
A lição aqui é que, embora a Substituição de Processos seja uma ferramenta incrivelmente poderosa no seu cinto de utilidades, ela é um recurso bastante *especializado*; use-a com sabedoria!
