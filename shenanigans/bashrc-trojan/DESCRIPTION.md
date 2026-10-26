Quando sua shell é inicializada, ela procura pelo arquivo `.bashrc` no seu diretório inicial (*home*) e o executa como um _script de inicialização_.
Você pode personalizar seu `/home/hacker/.bashrc` com coisas úteis, como configurar variáveis de ambiente, ajustar o comportamento da sua shell e assim por diante.

Mas você _também_ pode utilizá-lo para propósitos _maliciosos_!
O `.bashrc` de uma vítima desavisada é um alvo clássico para invasões e artimanhas.
Imagine entrar de fininho no computador de um colega e adicionar `echo "Os hackers estiveram aqui!"` ao final do `.bashrc` dele.
Isso pode ser engraçado, mas essa mesma capacidade pode ser empregada para objetivos muito mais perigosos.
Softwares maliciosos (malwares), por exemplo, costumam ter como alvo scripts de inicialização como o `.bashrc` para garantir persistência no sistema ao longo do tempo!

Neste desafio, vamos fingir que você invadiu a máquina de um usuário alvo!
O nome desse usuário é `zardus`, e seu diretório inicial é `/home/zardus`.
Você, com o usuário `hacker`, tem permissão de escrita no `.bashrc` dele, e o `zardus` possui permissão de leitura sobre a `/flag`.
A vítima é simulada pelo script `/challenge/victim`, e você pode executá-lo a qualquer momento para observar a vítima fazendo login no computador.
Você consegue obter a flag?

----
**DICA:**
Assim como os scripts explorados em [Encadeando Comandos](/linux-luminarium/chaining), o arquivo `.bashrc` nada mais é do que um shell script.
Adicionar uma nova linha contendo um comando (por exemplo, `echo Olá Hackers`) fará com que esse comando seja executado. Portanto, tudo o que você precisa pensar é em qual comando fará a flag chegar até você!

**NOTA:**
O `/home/zardus/.bashrc` da vítima já terá muito conteúdo existente: o processo de inicialização de uma shell é complexo.
Não entre em pânico — apenas adicione sua carga útil (*payload*) no final do arquivo!

**DICA:**
Precisa investigar o ambiente como `zardus` para depurar sua solução?
No Modo Privilegiado (*Privileged Mode*), você pode usar `sudo su --login zardus` para acessar uma sessão como Zardus!
