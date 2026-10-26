Vamos tentar a mesma coisa com o `tmux`!

O `tmux` (*terminal multiplexer*) é o primo mais novo e moderno do screen.
Ele faz as mesmas coisas, mas com atalhos de teclado um pouco diferentes.
A maior diferença?
Em vez de `Ctrl-A`, o tmux usa `Ctrl-B` como prefixo de comandos.

Portanto, para desconectar (`detach`) do tmux, pressione `Ctrl-B` seguido de `d`.

```console
hacker@dojo:~$ tmux
[doing some work...]
[Press Ctrl-B, then d]
[detached (from session 0)]
hacker@dojo:~$ 
```

Os comandos também variam:
- `tmux ls` - Lista as sessões
- `tmux attach` ou `tmux a` - Reconecta à sessão

Para este desafio:
1. Inicie o tmux
2. Desconecte-se dele (`detach`).
3. Execute `/challenge/run` (isso enviará a flag para sua sessão desconectada!)
4. Reconecte-se (`reattach`) para ver sua recompensa
