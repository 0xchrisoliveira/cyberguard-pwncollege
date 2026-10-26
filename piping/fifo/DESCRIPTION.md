Você aprendeu sobre pipes usando `|`, e viu que a substituição de processos cria pipes nomeados temporários (como `/dev/fd/63`).
Você também pode criar seus próprios pipes nomeados *persistentes*, que permanecem no sistema de arquivos até serem removidos!
Eles são chamados de **FIFOs**, sigla para *First (byte) In, First (byte) Out* (primeiro a entrar, primeiro a sair).

Você cria um FIFO usando o comando `mkfifo`:

```console
hacker@dojo:~$ mkfifo my_pipe
hacker@dojo:~$ ls -l my_pipe
prw-r--r-- 1 hacker hacker 0 Jan 1 12:00 my_pipe
hacker@dojo:~$ ls -l some_file
-rw-r--r-- 1 hacker hacker 0 Jan 1 12:00 some_file
hacker@dojo:~$
```

Repare na letra `p` no início da string de permissões — isso indica que se trata de um canal de pipe!
Isso difere claramente do `-` exibido no início de arquivos regulares, como `some_file` no exemplo acima.

Ao contrário dos pipes nomeados automáticos da substituição de processos:

- Você tem controle sobre o local onde os FIFOs são criados
- Eles persistem até que você os exclua explicitamente
- Qualquer processo pode ler ou escrever neles pelo caminho do arquivo (por exemplo, `echo hi > my_pipe`)
- Você pode visualizá-los com `ls` e inspecioná-los como arquivos comuns

Uma característica crucial dos FIFOs é que as operações neles realizam "bloqueio" (*blocking*) até que ambas as pontas (escritor e leitor) estejam conectadas e prontas.
Por exemplo, veja a situação:

```console
hacker@dojo:~$ mkfifo myfifo
hacker@dojo:~$ echo pwn > myfifo
```

Para atender a `echo pwn > myfifo`, o bash abrirá o arquivo `myfifo` em modo de escrita.
Contudo, essa operação ficará travada (aguardando) até que outro comando *também* abra o arquivo em modo de leitura (completando o circuito do pipe).
Isso pode ocorrer em outro terminal simultâneo:

```console
hacker@dojo:~$ cat myfifo
pwn
hacker@dojo:~$
```

O que aconteceu aqui?
Quando executamos `cat myfifo`, o pipe teve seus dois lados conectados e foi *desbloqueado*, permitindo que `echo pwn > myfifo` terminasse e transferisse o texto `pwn` através do pipe, sendo imediatamente lido pelo `cat`.

Naturalmente, arquivos comuns também conseguem trocar dados (você já usou `echo` para gravar e `cat` para ler).
Por que então usar um FIFO?
Veja as diferenças fundamentais:

1. **Sem armazenamento em disco:** Os FIFOs transferem dados diretamente entre processos na memória RAM — nada é gravado no disco físico.
2. **Dados efêmeros:** Assim que os dados são lidos de um FIFO, eles desaparecem do canal (diferente de arquivos regulares, onde o conteúdo permanece).
3. **Sincronização automática:** Processos de escrita bloqueiam até que haja um leitor pronto, e vice-versa. Isso fornece sincronização automática entre processos. No exemplo acima, não importa a ordem de disparo: se `cat myfifo` ou `echo pwn > myfifo` for executado primeiro, um esperará pacientemente pelo outro. Em arquivos normais, o leitor leria um arquivo vazio se executado antes da escrita terminar.
4. **Fluxos de dados complexos:** FIFOs são ideais para orquestrar fluxos de dados avançados, divisão e junção de canais, suportando múltiplos leitores e escritores.

Este desafio será uma introdução prática aos FIFOs.
Você precisará criar o FIFO em `/tmp/flag_fifo` e redirecionar a saída padrão (`stdout`) de `/challenge/run` para ele.
Se fizer tudo certo, `/challenge/run` enviará a flag para dentro do FIFO!
Mãos à obra!

----
**DICA:**
O comportamento bloqueante dos FIFOs pode tornar difícil resolver este desafio em um único terminal sem processos em segundo plano.
Pode ser muito prático abrir dois terminais (por exemplo, no modo Desktop ou VSCode) para ler e escrever simultaneamente no FIFO.
