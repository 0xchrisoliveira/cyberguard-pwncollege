À medida que você combina mais e mais comandos para produzir efeitos complexos, o tamanho da linha digitada no prompt rapidamente se torna cansativo de gerenciar.
Quando isso acontece, você pode colocar esses comandos dentro de um arquivo, chamado de *shell script*, e executá-los rodando esse arquivo!
Por exemplo, relembre a nossa técnica com ponto e vírgula:

```console
hacker@dojo:~$ echo COLLEGE > pwn; cat pwn
COLLEGE
hacker@dojo:~$
```

Podemos criar um shell script chamado `pwn.sh` (por convenção, scripts de shell costumam utilizar a extensão `.sh`):

```sh
echo COLLEGE > pwn
cat pwn
```

E então podemos executá-lo passando o arquivo como argumento para uma nova instância da nossa shell (`bash`)!
Quando uma shell é invocada dessa forma, em vez de esperar comandos digitados pelo usuário, ela lê e executa linha por linha os comandos do arquivo.

```console
hacker@dojo:~$ ls
hacker@dojo:~$ bash pwn.sh
COLLEGE
hacker@dojo:~$ ls
pwn
hacker@dojo:~$
```

Você pode notar que o shell script executou ambos os comandos, criando e imprimindo o arquivo `pwn`.

Agora é a sua vez!
Assim como no nível anterior, execute `/challenge/pwn` e depois `/challenge/college`, mas desta vez inserindo ambos em um shell script chamado `x.sh` e rodando-o com `bash`!

---
**NOTA:** Ainda não discutimos a ampla gama de excelentes editores de texto de linha de comando no Linux.
Por enquanto, sinta-se à vontade para usar o aplicativo `Text Editor` no modo Desktop (`Applications->Accessories->Text Editor`) ou o próprio editor do workspace no VSCode!
