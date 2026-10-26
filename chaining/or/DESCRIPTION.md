Você acabou de aprender sobre o operador `&&`, que executa o segundo comando apenas se o primeiro for bem-sucedido.
Agora, vamos conhecer o seu oposto: o operador `||` permite executar o segundo comando apenas se o primeiro comando falhar (isto é, terminar com um código de saída diferente de zero).
Ele é chamado de operador "OR" (OU lógico) porque ou o primeiro comando tem sucesso OU o segundo comando será executado como alternativa.

Veja a sintaxe:
```console
hacker@dojo:~$ command1 || command2
```

Isso significa: "Execute command1 e, SE ele falhar, execute command2."

Alguns exemplos:
```console
hacker@dojo:~$ touch /file || echo "touch failed, so this runs"
touch: cannot touch '/file': Permission denied
touch failed, so this runs
hacker@dojo:~$ touch /home/hacker/file || echo "this will NOT run"
hacker@dojo:~$
```

O operador `||` é muito útil para fornecer comandos alternativos de contingência (*fallback*) ou tratamento básico de erros!

Neste desafio, você precisa encadear `/challenge/first-failure` e `/challenge/second` utilizando o operador `||`.
Manda ver!
