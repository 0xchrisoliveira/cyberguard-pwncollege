O espaço disponível em `/home/hacker` neste contêiner é de míseros **1 gigabyte**.
Neste nível, você vai entupir o diretório `/home/hacker` com tanto lixo que nem mesmo um pequeno arquivo de 1 megabyte conseguirá ser criado.
Quando isso ocorre, o ambiente de trabalho torna-se inoperante.
Vamos praticar a indução dessa falha neste desafio e entender as repercussões disso.

Como encher o disco?
Existem incontáveis maneiras.
Aqui, ensinaremos o comando `yes`!

```
hacker@dojo:~$ yes | head
y
y
y
y
y
y
y
y
y
y
hacker@dojo:~$
```

O comando `yes` gera uma saída contínua com a letra `y` repetidamente para sempre.
O uso convencional dele é automatizar respostas afirmativas em solicitações de confirmação ("Tem certeza de que deseja excluir este arquivo?") via pipes, mas aqui o utilizaremos para gerar um arquivo gigantesco repleto de linhas "y".
Basta redirecionar a saída do `yes` para um arquivo no seu diretório home e o seu disco estará cheio em um ou dois minutos!

Este desafio exige que você lote o disco e, em seguida, faça a limpeza.
O passo a passo:

1. Encha o disco.
2. Execute `/challenge/check`. Ele tentará criar um arquivo temporário de 1 megabyte. Se essa criação falhar, você passará da primeira etapa e o verificador pedirá que você libere o espaço.
3. Exclua o arquivo que você criou (usando `rm`) para liberar o espaço ocupado.
4. Execute `/challenge/check` uma segunda vez. Se agora ele conseguir criar o arquivo temporário (ou seja, se você limpou seu diretório home com sucesso), você receberá a flag.

----
**NOTA:**
Para este desafio, `/home/hacker` é um diretório home temporário e reduzido.
Seus arquivos habituais do diretório home não estarão visíveis aqui, e arquivos salvos aqui não persistirão após a conclusão do desafio.

**Por que duas etapas?**
Seu diretório home persiste entre as instâncias dos desafios.
Se deixássemos o disco permanentemente cheio, seu ambiente no pwn.college deixaria de funcionar.
Essa é _de longe_ a causa mais frequente de problemas estranhos relatados no pwn.college!

**SOCORRO, QUEBROU TUDO!**
Se você encher o disco e não limpá-lo depois, precisará entrar via `ssh` para consertar (removendo o arquivo volumoso).
Isso exige um pouco mais de cuidado, mas explicamos como fazer na seção "Conectando via SSH" do módulo [Começando](/welcome/welcome).
