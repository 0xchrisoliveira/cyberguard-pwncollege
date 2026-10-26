Certo, o Zardus ficou esperto mais uma vez!
Nada de compartilhar o diretório inicial: apesar da menor conveniência, Zardus passou a compartilhar `/tmp/collab`.
Ele tornou esse diretório gravável por todos e começou a criar uma lista de comandos perigosos para memorizar!

```console
zardus@dojo:~$ mkdir /tmp/collab
zardus@dojo:~$ chmod a+w /tmp/collab
zardus@dojo:~$ echo "rm -rf /" > /tmp/collab/evil-commands.txt
```

Neste desafio, quando você executa `/challenge/victim`, Zardus adiciona `cat /flag` a essa lista de comandos:

```console
hacker@dojo:~$ /challenge/victim

Username: zardus
Password: **********
zardus@dojo:~$ echo "cat /flag" >> /tmp/collab/evil-commands.txt
zardus@dojo:~$ exit
logout

hacker@dojo:~$
```

Lembre-se do nível anterior que, com permissão de escrita em `/tmp/collab`, o usuário `hacker` pode substituir o arquivo `evil-commands.txt`.
Lembre-se também de [Compreendendo Comandos](/linux-luminarium/commands) que arquivos podem criar _links_ para outros arquivos.
O que acontece se o usuário `hacker` substituir `evil-commands.txt` por um link simbólico apontando para algum arquivo sensível no qual `zardus` pode escrever?
Caos e artimanhas puras!

Você _sabe_ para qual arquivo deve apontar o link.
Execute o ataque e capture a `/flag` (que, neste nível, Zardus pode ler novamente!).

----
**DICA:**
Você precisará executar `/challenge/victim` duas vezes: uma para que `cat /flag` seja gravado no destino desejado, e outra para dispará-lo!

**O `/tmp` é perigoso de usar???**
Apesar do ataque demonstrado aqui, o `/tmp` pode ser utilizado com segurança.
O diretório _é gravável por todos_ (*world-writable*), mas possui um bit especial de permissão habilitado:

```console
hacker@dojo:~$ ls -ld /tmp
drwxrwxrwt 29 root root 1056768 Jun  6 14:06 /tmp
hacker@dojo:~$
```

Essa letra `t` no final representa o bit _sticky_ (*sticky bit*).
O sticky bit determina que o diretório apenas permita que os proprietários dos arquivos possam renomear ou remover arquivos dentro dele.
Ele foi projetado especificamente para impedir esse tipo de ataque!
O problema neste desafio, claramente, foi que Zardus não habilitou o sticky bit em `/tmp/collab`.
Fazer isso teria eliminado a vulnerabilidade neste caso específico:

```console
zardus@dojo:~$ chmod +t /tmp/collab
```

Evidentemente, recursos compartilhados como diretórios graváveis por qualquer usuário continuam sendo arriscados.
Muito mais adiante, no módulo de [Condições de Corrida (Race Conditions)](/system-security/race-conditions) do conteúdo de Faixa Verde (*Green Belt*), você verá diversas formas em que esses recursos podem desencadear problemas de segurança!
