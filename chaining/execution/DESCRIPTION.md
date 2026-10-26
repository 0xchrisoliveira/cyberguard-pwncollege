Você escreveu seus primeiros shell scripts, mas ter que chamá-los digitando `bash script.sh` toda vez pode ser inconveniente.
Por que você precisaria desse `bash` explícito afinal?

Quando você executa `bash script.sh`, você está, na verdade, iniciando o utilitário `bash` com o argumento `script.sh`.
Isso avisa ao bash para ler os comandos do arquivo `script.sh` em vez da entrada padrão (*stdin*), e assim o seu script é executado.

Acontece que você pode dispensar a necessidade de invocar o `bash` manualmente.
Se o seu arquivo de script for *executável* (relembre o módulo de [Permissões de Arquivos](/linux-luminarium/permissions)), você pode simplesmente chamá-lo diretamente através do seu caminho relativo ou absoluto!
Por exemplo, se você criar um arquivo `script.sh` no seu diretório pessoal (*home*) *e torná-lo executável*, poderá executá-lo com `/home/hacker/script.sh`, `~/script.sh` ou (se o seu diretório atual for `/home/hacker`) `./script.sh`.

Experimente isso aqui!
Crie um shell script que execute `/challenge/solve`, torne o arquivo executável e execute-o diretamente sem invocar explicitamente o `bash`!
