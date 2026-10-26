Assim como você pode redirecionar a *saída* de programas, também pode redirecionar a *entrada* para dentro dos programas!
Isso é feito com o operador `<`, como no exemplo:

```
hacker@dojo:~$ echo yo > message
hacker@dojo:~$ cat message
yo
hacker@dojo:~$ rev < message
oy
```

O redirecionamento de entrada abre inúmeras possibilidades interessantes com diversos programas!
Neste nível, praticaremos com `/challenge/run`, que exigirá que você redirecione o arquivo `PWN` para sua entrada padrão, e que esse arquivo `PWN` contenha exatamente o valor `COLLEGE`!
Para gravar esse valor no arquivo `PWN`, lembre-se do que aprendeu nos desafios anteriores sobre redirecionamento de saída com `echo`!
