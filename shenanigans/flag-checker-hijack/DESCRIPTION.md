No nível anterior, você explorou o `~/.bashrc` do Zardus para fazê-lo executar comandos em seu favor.

Desta vez, o Zardus não deixa a flag em um arquivo legível após fazer login.
Em vez disso, ele executará um comando chamado `flag_checker`, digitando a flag manualmente nele para verificação.

Sua missão é utilizar o seu acesso contínuo de escrita ao `.bashrc` do Zardus para interceptar essa flag.
Lembra-se de como você sequestrou comandos no módulo [Desvendando o PATH](/linux-luminarium/path)?
Consegue usar essa mesma técnica para interceptar o `flag_checker`?

----
**DICA:**
O Zardus está desconfiando da sua interceptação?
Ele é cauteloso — ele verifica se a mensagem de solicitação (*prompt*) do `flag_checker` exibe exatamente `Type the flag`.
Certifique-se de que o seu binário interceptador também imprima esse prompt (por exemplo, `echo "Type the flag"`).
Além de imprimir esse prompt, o seu `flag_checker` falso pode:
a) Simplesmente passar a entrada do Zardus para o stdout com `cat` (usando `cat` sem argumentos), ou
b) Ler a entrada em uma variável com `read` e exibi-la com `echo`.
A escolha é sua!

----
**DICA:**
Não se esqueça de tornar o seu `flag_checker` falso executável, conforme você aprendeu no módulo [Compreendendo Permissões](/linux-luminarium/permissions)!
