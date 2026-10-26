Arquivos (ou saídas de comandos) nem sempre aparecem na ordem em que precisamos!
O comando `sort` ajuda a organizar esses fluxos de dados.
Ele lê as linhas de uma entrada padrão (ou de arquivos) e as exibe devidamente ordenadas:

```console
hacker@dojo:~$ cat names.txt
  hack
  the
  planet
  with
  pwn
  college
hacker@dojo:~$ sort names.txt
  college
  hack
  planet
  pwn
  the
  with
hacker@dojo:~$
```

Por padrão, o `sort` organiza as linhas em ordem alfabética.
Várias opções permitem modificar essa ordenação:

- `-r`: ordem reversa (de Z a A)
- `-n`: ordenação numérica (interpreta valores como números)
- `-u`: apenas linhas únicas (*unique*, descarta duplicatas)
- `-R`: ordenação aleatória (*random*)!

Neste desafio, existe um arquivo em `/challenge/flags.txt` com 100 flags falsas e a flag verdadeira escondida no meio delas.
Ao ordenar o arquivo alfabeticamente, a flag real ficará na última linha (garantimos isso na geração dos despistes).
Vá até lá e capture-a!
