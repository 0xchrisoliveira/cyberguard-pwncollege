Se você utilizar o Linux (ou computadores em geral) por algum tempo para realizar trabalhos reais, você inevitavelmente se deparará com a seguinte situação: você precisa que dois programas acessem os mesmos dados, mas cada um espera encontrar esses dados em um local diferente.
Felizmente, o Linux oferece uma solução elegante para esse dilema: os _links_ (ligações).

Existem dois tipos principais de links: links _físicos_ (*hard links*) e links _simbólicos_ (*soft* ou *symbolic links*).
Podemos diferenciá-los com uma analogia simples:

- Um **hard link** é como usar formas alternativas para indicar o endereço do seu apartamento que chegam diretamente ao mesmo lugar (por exemplo: "Apartamento 2" versus "Unidade 2").
- Um **soft link** (link simbólico) é como quando você se muda de casa e pede aos correios que redirecionem automaticamente todas as cartas do seu endereço antigo para o endereço novo.

No sistema de arquivos, um arquivo é conceitualmente um endereço onde o conteúdo de dados reside.
Um *hard link* é um endereço alternativo que indexa diretamente esses mesmos dados — acessos pelo hard link ou pelo arquivo original são absolutamente idênticos e trazem os dados imediatamente.
Já um *soft link* (link simbólico) armazena, em vez disso, o caminho do arquivo original.
Quando você acessa o link simbólico, o Linux reconhece que se trata de um atalho simbólico, lê o nome/caminho original apontado e então (geralmente) acessa aquele arquivo automaticamente.
Na grande maioria dos casos, ambos os métodos permitem acessar os dados originais, mas a mecânica por baixo dos panos é diferente.

Hard links podem parecer mais simples à primeira vista, mas possuem diversas restrições e particularidades que tornam os links simbólicos (*symlinks*), de longe, a alternativa mais comum e flexível no dia a dia.

Neste desafio, aprenderemos sobre links simbólicos (*symlinks*).
Eles são criados com o comando `ln` utilizando o modificador `-s` na sintaxe `ln -s ALVO NOME_DO_LINK`.
`ALVO` (*TARGET*) é o caminho para onde o link deve apontar, e `NOME_DO_LINK` (*LINK_NAME*) é o novo caminho/atalho que você está criando.
Por exemplo:

```console
hacker@dojo:~$ cat /tmp/myfile
This is my file!
hacker@dojo:~$ ln -s /tmp/myfile /home/hacker/ourfile
hacker@dojo:~$ cat ~/ourfile
This is my file!
hacker@dojo:~$
```

Você pode notar que acessar o symlink entrega exatamente o conteúdo do arquivo original!
Neste exemplo, `/tmp/myfile` é o alvo e `/home/hacker/ourfile` é o nome do link.

Um link simbólico pode ser identificado de várias maneiras.
Por exemplo, o comando `file`, que analisa um arquivo e informa o seu tipo, reconhece links simbólicos perfeitamente:

```console
hacker@dojo:~$ file /tmp/myfile
/tmp/myfile: ASCII text
hacker@dojo:~$ file ~/ourfile
/home/hacker/ourfile: symbolic link to /tmp/myfile
hacker@dojo:~$
```

Tudo pronto, agora é sua vez de testar!
Neste nível, a flag está, como sempre, em `/flag`, mas o programa `/challenge/catflag` tentará ler o arquivo `/home/hacker/not-the-flag`.
O caminho `/home/hacker/not-the-flag` é o nome do link que você deve criar neste desafio.
Escolha o alvo correto que fará o `/challenge/catflag` ser enganado e entregar a flag a você!

----
**ATENÇÃO:** Se algum experimento anterior deixou algum arquivo existente em `/home/hacker/not-the-flag`, remova-o primeiro com `rm` para que o comando `ln` possa criar o link simbólico no local.
