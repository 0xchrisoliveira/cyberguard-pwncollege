Sem nenhum argumento, o comando `su` iniciará uma shell como `root` (após a autenticação com a senha do `root`).
No entanto, você também pode passar um nome de usuário como argumento para alternar para *aquele* usuário específico em vez do `root`.
Por exemplo:

```console
hacker@dojo:~$ su some-user
Password:
some-user@dojo:~$
```

Sensacional!
Neste nível, você deve alternar para o usuário `zardus` e então executar `/challenge/run`.
A senha de zardus é `dont-hack-me`.
Boa sorte!
