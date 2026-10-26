Embora a maioria dos comandos seja simples de usar, a sintaxe de alguns utilitários pode ser bastante sofisticada.
Por exemplo, os argumentos passados para ferramentas como `sed` e `awk` (que não abordaremos no momento) são praticamente programas inteiros escritos em linguagens próprias!
Em algum ponto no espectro entre o simples `cd` e o complexo `awk`, existem comandos que recebem argumentos para os seus próprios argumentos...

Isso pode soar estranho no início, mas você já presenciou isso no nível do `find` em [Compreendendo Comandos](/linux-luminarium/commands).
O `find` aceita o argumento `-name`, e a opção `-name`, por sua vez, exige um argumento próprio indicando o padrão de nome a ser buscado.
Muitos outros comandos funcionam de maneira análoga.

Veja a documentação deste nível para `/challenge/challenge`:

> Bem-vindo à documentação do `/challenge/challenge`!
> Este programa exibe o conteúdo de arquivos arbitrários no terminal quando recebe o argumento `--printfile`.
> O argumento para `--printfile` é o caminho do arquivo a ser lido.
> Por exemplo, `/challenge/challenge --printfile /etc/hostname` exibirá o hostname do container!

Com base nessa documentação, vá e capture a flag!
