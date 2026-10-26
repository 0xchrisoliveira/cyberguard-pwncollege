Vamos aprender a navegar pelas janelas no tmux!

Assim como o screen, o tmux possui janelas.
As combinações de teclas são diferentes, mas o conceito é o mesmo:

- `Ctrl-B c` - Cria uma nova janela
- `Ctrl-B n` - Próxima janela (*Next*)
- `Ctrl-B p` - Janela anterior (*Previous*)
- `Ctrl-B 0` até `Ctrl-B 9` - Pula diretamente para as janelas 0 a 9
- `Ctrl-B w` - Exibe uma interface visual para seleção de janelas

O tmux exibe suas janelas na parte inferior em uma barra de status parecida com:
```
[0] 0:bash* 1:bash
```

O asterisco `*` indica a sua janela atual, e cada entrada também mostra o processo que a janela foi criada para executar.

Criamos uma sessão do tmux com duas janelas:
- A Janela 0 contém a flag!
- A Janela 1 contém as boas-vindas.

Vá em frente e capture essa flag!
