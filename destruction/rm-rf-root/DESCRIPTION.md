Quer apagar tudo e recomeçar do zero?
Você pode!

```console
hacker@dojo:~$ ls /
bin etc blah blah blah
hacker@dojo:~$ rm -rf /
hacker@dojo:~$ ls /
bash: ls: command not found
hacker@dojo:~$
```

O que aconteceu aqui?
Como você se lembra, `rm` remove arquivos.
A opção `-r` (*recursive*) remove diretórios e todos os arquivos contidos neles.
A opção `-f` (*force*) ignora quaisquer erros ou avisos de confirmação que o `rm` encontraria.
Combinadas e apontadas para a raiz `/`, os resultados são catastróficos: uma exclusão total do seu sistema operacional.
Em um sistema moderno, as coisas não são _tão_ simples assim de primeira, mas você entenderá assim que colocar em prática.

Neste desafio, você fará algo que talvez nunca mais faça: aniquilar todo o sistema.
Nós fizemos um pequeno ajuste para proteger o seu diretório pessoal (normalmente ele seria varrido junto!), mas, tirando isso, tudo o que separa você da flag é a sua coragem de deletar a unidade inteira.
Mas antes de apagar tudo, certifique-se de executar `/challenge/check` em outro terminal para que ele possa assistir ao espetáculo (e entregar a flag para você)!

----
**NOTA:**
O comando `rm` levará algum tempo para concluir.
Há muita coisa para deletar!

**NOTA:**
Existem diversos motivos técnicos pelos quais é improvável que você consiga deletar absolutamente _tudo_, incluindo a técnica que utilizamos para proteger seu diretório home neste nível.
Não se preocupe: você causará estragos mais do que suficientes!
