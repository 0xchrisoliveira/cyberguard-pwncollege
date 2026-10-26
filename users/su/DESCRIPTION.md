Não são apenas invasores que precisam se tornar `root`.
Muitas vezes, você, como dono do seu próprio computador, precisa de acesso de `root` para administrá-lo.
Tornar-se `root` é uma ação bastante comum entre usuários de Linux, e existem dois utilitários principais para essa finalidade: `su` e `sudo`.

Neste desafio, abordaremos o mais antigo deles: `su` (comando para **s**ubstituir **u**suário — *substitute user*).
Ele não é mais tão comumente utilizado no dia a dia moderno para elevar privilégios a `root`, mas é um utilitário elegante e clássico que cobriremos primeiro.

O `su` é um binário com bit setuid:

```console
hacker@dojo:~$ ls -l /usr/bin/su
-rwsr-xr-x 1 root root 232416 Dec 1 11:45 /usr/bin/su
hacker@dojo:~$
```

Como possui o bit SUID ativado, o `su` executa com os privilégios de `root`.
Rodando como `root`, ele tem permissão para iniciar uma shell de `root`!
Claro que o `su` não é ingênuo: antes de permitir que o usuário eleve seus privilégios para `root`, ele verifica se o usuário sabe a senha do `root`:

```console
hacker@dojo:~$ su
Password: 
su: Authentication failure
hacker@dojo:~$
```

Essa checagem contra uma senha global de `root` é justamente o que tornou o `su` menos comum em ambientes modernos.
Sistemas modernos raramente definem senhas diretas de `root`, preferindo outros mecanismos (que veremos a seguir) para conceder privilégios administrativos.

Mas ESTE desafio (e somente este desafio) possui, sim, uma senha de `root`.
Essa senha é `hack-the-planet`, e você deve fornecê-la ao `su` para se tornar `root`!
Faça isso agora e leia a flag!
