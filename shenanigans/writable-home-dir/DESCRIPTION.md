Muito bem, Zardus ficou mais esperto — afinal de contas, por que ele deixaria o `.bashrc` gravável por qualquer um?
Porém, um cenário muito mais comum é usuários no mesmo sistema, a fim de facilitar a colaboração, tornarem seus diretórios pessoais (*home*) _graváveis por todos_ (*world-writable*).
Qual é o grande perigo disso?

O problema reside em uma sutileza das permissões de arquivos/diretórios no Linux: qualquer pessoa com permissão de escrita em um diretório pode _mover_ e _excluir_ arquivos dentro dele.
Por exemplo, suponha que Zardus crie um diretório com permissão total de escrita para colaboração:

```console
zardus@dojo:~$ mkdir /tmp/collab
zardus@dojo:~$ chmod a+w /tmp/collab
zardus@dojo:~$ echo "do pwn.college" > /tmp/collab/todo-list
```

Em seguida, um invasor surge e executa o seguinte, _mesmo não sendo o dono do arquivo todo-list_!

```console
hacker@dojo:~$ ls -l /tmp/collab/todo-list
-rw-r--r-- 1 zardus zardus 15 Jun  6 13:12 /tmp/collab/todo-list
hacker@dojo:~$ rm /tmp/collab/todo-list
rm: remove write-protected regular file '/tmp/collab/todo-list'? y
hacker@dojo:~$ echo "send hacker money" > /tmp/collab/todo-list
hacker@dojo:~$ ls -l /tmp/collab/todo-list
-rw-r--r-- 1 hacker hacker 18 Jun  6 13:12 /tmp/collab/todo-list
hacker@dojo:~$
```

Isso pode parecer contra-intuitivo à primeira vista: o `hacker` não tinha permissão de escrita no arquivo `todo-list`, mas o resultado final foi que ele conseguiu substituir todo o seu conteúdo.
Pense da seguinte forma: o vínculo de um arquivo com um diretório reside, em última instância, no próprio diretório. Assim, usuários com permissão de escrita naquele diretório podem alterar essas ligações.
Evidentemente, isso acarreta sérias implicações de segurança quando diretórios importantes estão com permissão de escrita para qualquer um.

Neste desafio, por pura conveniência, Zardus abriu as permissões do seu diretório inicial:

```console
zardus@dojo:~$ chmod a+w /home/zardus
```

Como você sabe, existem muitos arquivos críticos nesse diretório, _como o `.bashrc`_!
Você consegue reproduzir o ataque anterior tendo permissão de escrita em `/home/zardus`, em vez de ter acesso direto a `/home/zardus/.bashrc`?
