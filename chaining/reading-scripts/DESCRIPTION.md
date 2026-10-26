Você não é o único que escreve shell scripts!
Eles são extremamente práticos para tarefas rotineiras a nível de sistema e constituem uma das ferramentas mais comuns para desenvolvedores e administradores de sistemas (sysadmins).
Na verdade, uma fração surpreendente dos programas utilitários presentes em uma máquina Linux convencional são, sob o capô, scripts em shell.

Neste nível, aprenderemos a ler shell scripts.
O executável `/challenge/run` é um script em shell que exige a digitação de uma senha secreta, mas essa senha está gravada diretamente no próprio código-fonte do script (*hardcoded*)!
Leia o script (usando, por exemplo, o comando `cat`), identifique a senha e capture a sua flag!

----
**NOTA:**
Sinta-se encorajado a tentar ler o código de outros desafios também!
Ler código é uma estratégia fundamental para desenvolver novas habilidades, pois permite observar como certas funcionalidades foram construídas e reaproveitar essas soluções em seus próprios scripts.
Mas atenção: alguns arquivos de programas são binários compilados em código de máquina e não serão legíveis para humanos.
Você pode usar o comando `file` para distinguir binários compilados de scripts de texto, mas quase todos os desafios do Linux Luminarium são implementados como shell scripts e podem ser lidos tranquilamente com o `cat`.
