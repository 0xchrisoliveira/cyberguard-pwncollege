Às vezes, você precisa comparar a saída de dois comandos em vez de dois arquivos pré-existentes.
Sua primeira ideia poderia ser salvar a saída de cada um em um arquivo temporário:

```console
hacker@dojo:~$ command1 > file1
hacker@dojo:~$ command2 > file2
hacker@dojo:~$ diff file1 file2
```

Mas há uma maneira muito mais elegante! O Linux segue a consagrada filosofia de que ["tudo é um arquivo"](https://en.wikipedia.org/wiki/Everything_is_a_file).
Ou seja, o sistema busca oferecer uma interface semelhante a arquivos para a maioria dos recursos, inclusive para a entrada e a saída de programas em execução!
A shell reflete diretamente essa filosofia, permitindo, por exemplo, que utilitários que esperam caminhos de arquivos como argumentos recebam diretamente a saída de processos.

Podemos ir além e conectar a entrada e saída de programas aos *argumentos* dos comandos.
Isso é feito usando a [Substituição de Processos](https://www.gnu.org/software/bash/manual/html_node/Process-Substitution.html) (*Process Substitution*).
Para ler a saída de um comando como se fosse um arquivo de entrada, usamos `<(comando)`.
Ao escrever `<(comando)`, o bash executa o comando e conecta sua saída a um arquivo temporário especial.
Esse não é um arquivo comum gravado no disco; trata-se de um *pipe nomeado* (*named pipe*), que possui um caminho identificável no sistema:

```console
hacker@dojo:~$ echo <(echo hi)
/dev/fd/63
hacker@dojo:~$
```

De onde surgiu esse `/dev/fd/63`?
O `bash` substituiu `<(echo hi)` pelo caminho do pipe nomeado conectado à saída do comando!
Enquanto o comando estiver em execução, qualquer leitura desse arquivo lerá diretamente os dados da saída padrão do processo.
Isso é frequentemente usado com comandos que aceitam arquivos como argumentos:

```console
hacker@dojo:~$ cat <(echo hi)
hi
hacker@dojo:~$
```

E, é claro, você pode utilizar essa substituição múltiplas vezes no mesmo comando:

```console
hacker@dojo:~$ echo <(echo pwn) <(echo college)
/dev/fd/63 /dev/fd/64
hacker@dojo:~$ cat <(echo pwn) <(echo college)
pwn
college
hacker@dojo:~$
```

Agora, o seu desafio!
Lembre-se do que aprendeu sobre o comando `diff` nos módulos anteriores.
Naquela ocasião, você comparou dois arquivos físicos.
Agora, você comparará a saída de dois programas em tempo real: `/challenge/print_decoys`, que imprime várias flags falsas (*decoys*), e `/challenge/print_decoys_and_flag`, que imprime os mesmos despistes acrescidos da flag verdadeira.

Use substituição de processo com `diff` para comparar as saídas desses dois programas diretamente e capturar a flag!
