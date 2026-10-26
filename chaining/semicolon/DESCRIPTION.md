A maneira mais simples de encadear comandos é com o ponto e vírgula: `;`.
Na maioria dos contextos, o `;` separa comandos de forma semelhante a como a tecla Enter separa linhas.
Portanto, isto:

```console
hacker@dojo:~$ echo COLLEGE > pwn
hacker@dojo:~$ cat pwn
COLLEGE
hacker@dojo:~$
```

É praticamente o mesmo que isto:

```console
hacker@dojo:~$ echo COLLEGE > pwn; cat pwn
COLLEGE
hacker@dojo:~$
```

Basicamente, quando você pressiona Enter, a shell executa o comando digitado e, após o término desse comando, exibe o prompt novamente para você digitar o próximo.
O ponto e vírgula é análogo, com a diferença de que não há um novo prompt entre eles e você insere ambos os comandos antes que qualquer um deles seja executado.

Experimente agora! Neste nível, você deve executar `/challenge/pwn` e, em seguida, `/challenge/college`, encadeando-os com um ponto e vírgula.
