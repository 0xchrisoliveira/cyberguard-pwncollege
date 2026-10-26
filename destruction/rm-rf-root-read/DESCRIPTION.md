Vamos analisar os efeitos de varrer do mapa todo o seu sistema de arquivos.
Você agora é um especialista no uso de `rm`, mas no desafio anterior o `/challenge/check` imprimiu a flag na tela assim que o disco foi limpo.
E se ele não fizesse isso?
Sem o binário `cat`, como você conseguiria ler a `/flag`?

Lembre-se, do módulo [Explorando a Documentação](/linux-luminarium/man), de que alguns comandos da shell são recursos embutidos (*builtins*).
Enquanto `ls`, `cat` e programas semelhantes são binários externos, o comando `read` (que, como visto no módulo [Variáveis de Shell](/linux-luminarium/variables), consegue ler arquivos!) _é_ um comando embutido da shell.
Isso significa que, mesmo se você destruir todo o sistema de arquivos, desde que você mantenha uma instância do bash já aberta e rodando, ainda será possível ler arquivos!

Este desafio colocará isso à prova.
Ele é muito parecido com o anterior, mas você precisará ler a flag por conta própria após aniquilar o sistema.
Depois que você remover tudo com `rm`, o `/challenge/check` (que deve ter sido iniciado previamente) restaurará o arquivo `/flag` com permissão de leitura.
Depois disso, bastará usar `read` para capturá-la!
