Você aprendeu sobre códigos de saída no módulo de [Processos](https://pwn.college/linux-luminarium/processes/).
Agora, vamos usá-los para encadear comandos!

O operador `&&` permite que você execute um segundo comando apenas se o primeiro comando for bem-sucedido (na convenção do Linux, isso significa que ele finalizou com código de saída 0).
Ele é chamado de operador "AND" (E lógico) porque ambas as condições precisam ser satisfeitas: o primeiro comando deve ter sucesso E só então o segundo comando será executado.
Isso é extremamente útil em fluxos de trabalho avançados na linha de comando onde determinadas ações dependem do êxito de etapas anteriores.

Veja a sintaxe:
```console
hacker@dojo:~$ command1 && command2
```

Isso significa: "Execute command1 e, SE ele for bem-sucedido, execute command2."

Alguns exemplos:

```console
hacker@dojo:~$ touch /home/hacker/file && echo "this will run"
success
this will run
hacker@dojo:~$ touch /file && echo "this will NOT run"
touch: cannot touch '/file': Permission denied
hacker@dojo:~$
```

A segunda invocação de `touch` falhou porque o usuário hacker não tem permissão de escrita em `/file`; portanto, o comando `echo` não foi executado.

Neste desafio, você precisa encadear os programas `/challenge/first-success` e `/challenge/second` utilizando o operador `&&`.
Experimente executar cada comando separadamente primeiro para ver o que acontece (você verá que *não* receberá a flag).
Mas se encadeá-los com `&&`, a flag aparecerá!
