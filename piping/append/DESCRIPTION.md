Um caso de uso muito comum do redirecionamento de saída é armazenar o resultado de comandos para análise posterior.
Frequentemente, você deseja fazer isso de forma *agregada*: executar vários comandos sucessivos, acumular suas saídas e depois pesquisar nelas com `grep`.
Nesse cenário, você precisa que a nova saída seja sempre adicionada ao final do mesmo arquivo, enquanto o operador `>` cria um arquivo novo a cada execução, sobrescrevendo o conteúdo anterior.

Você pode redirecionar a saída em modo de *anexo* (*append mode*) utilizando `>>` em vez de `>`, assim:

```console
hacker@dojo:~$ echo pwn > outfile
hacker@dojo:~$ echo college >> outfile
hacker@dojo:~$ cat outfile
pwn
college
hacker@dojo:$
```

Para praticar, execute `/challenge/run` redirecionando a saída em modo de anexo (*append mode*) para o arquivo `/home/hacker/the-flag`.
O desafio gravará a primeira metade da flag no arquivo e tentará enviar a segunda metade para o `stdout` se o `stdout` estiver sendo redirecionado para o arquivo.
Se você redirecionar corretamente com `>>`, a segunda metade será anexada à primeira. Porém, se usar o modo de truncamento (`>`), a segunda metade *sobrescreverá* a primeira e a flag ficará incompleta!

Mãos à obra!
