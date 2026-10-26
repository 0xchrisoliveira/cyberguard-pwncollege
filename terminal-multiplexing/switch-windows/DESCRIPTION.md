Até agora, o `screen` parece apenas um tipo curioso de terminal dentro de outro terminal.
Mas ele pode ser muito mais!

Dentro de uma única sessão do screen, você pode ter várias janelas, assim como o seu navegador tem várias abas.
Isso é extremamente prático para organizar tarefas diferentes!

Essas janelas são gerenciadas por meio de atalhos de teclado, todos iniciando com `Ctrl-A`:

- `Ctrl-A c` - Cria uma nova janela
- `Ctrl-A n` - Próxima janela (*Next*)
- `Ctrl-A p` - Janela anterior (*Previous*)
- `Ctrl-A 0` até `Ctrl-A 9` - Pula diretamente para as janelas 0 a 9
- `Ctrl-A "` - Abre um menu de seleção com todas as janelas

Para este desafio, preparamos uma sessão do screen com duas janelas:

- A Janela 0 tem... bem, você terá que alternar para lá para descobrir!
- A Janela 1 contém uma mensagem de boas-vindas

Conecte-se à sessão com `screen -r` e, em seguida, use uma das combinações de teclas acima para alternar para a outra janela.
Vá em frente e capture essa flag!
