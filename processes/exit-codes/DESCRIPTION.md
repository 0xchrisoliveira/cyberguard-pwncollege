Todo comando na shell, incluindo cada programa executável e cada comando interno (*builtin*), retorna um *código de saída* (*exit code* ou *return code*) ao terminar sua execução.
Isso pode ser usado pela shell ou pelo próprio usuário (você!) para verificar se o processo concluiu com sucesso a sua tarefa (essa interpretação depende, claro, do propósito original do comando).

Você pode acessar o código de saída do último comando encerrado utilizando a variável especial `?` (não se esqueça de prefixá-la com `$` para ler o seu valor: `$?`):

```console
hacker@dojo:~$ touch test-file
hacker@dojo:~$ echo $?
0
hacker@dojo:~$ touch /test-file
touch: cannot touch '/test-file': Permission denied
hacker@dojo:~$ echo $?
1
hacker@dojo:~$
```

Como você pode ver, comandos bem-sucedidos normalmente retornam `0`, enquanto comandos que falham retornam um valor diferente de zero — comumente `1`, mas por vezes um código específico que identifica o tipo de erro ocorrido.

Neste desafio, você deve capturar o código de saída retornado por `/challenge/get-code` e, em seguida, executar `/challenge/submit-code` passando esse código como argumento.
Boa sorte!
