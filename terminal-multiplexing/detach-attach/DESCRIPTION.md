Agora vamos começar a explorar a mágica do _desanexamento_ (_detaching_)!

Imagine que você esteja trabalhando em algo importante em uma conexão remota e sua conexão caia.
Com um terminal comum (fora deste ambiente incrível do dojo), tudo seria perdido.
Com o screen, seu trabalho continua rodando em segundo plano e você pode _reconectá-lo_ (_reattach_) mais tarde!

Você também pode se _desconectar_ (_detach_) de propósito, que é o que faremos neste desafio.
Você se desconecta pressionando `Ctrl-A`, seguido de `d` (de **d**etach).
Isso mantém sua sessão rodando em segundo plano enquanto você retorna ao terminal normal.

```console
hacker@dojo:~$ screen
[doing some work...]
[Press Ctrl-A, then d]
[detached from 12345.pts-0.hostname]
hacker@dojo:~$ 
```

Para se **r**econectar (**r**eattach), você pode usar o argumento `-r` no `screen`:

```console
hacker@dojo:~$ screen -r
```

Para este desafio, você precisará:

1. Iniciar o screen
2. Desconectar-se dele (`detach`).
3. Executar `/challenge/run` (isso enviará a flag secretamente para sua sessão desconectada!)
4. Reconectar-se (`reattach`) para ver sua recompensa

----
**CURIOSIDADE:**
`Ctrl-A` é a tecla de ativação padrão do `screen` para todos os seus atalhos.
Toda a funcionalidade do `screen` é acionada por alguma combinação de comandos iniciada com `Ctrl-A`.

**DICA:**
Lembre-se: segure Ctrl e aperte A, depois solte ambos e aperte d.

**DICA:**
Se você vir `[detached from...]`, você fez certinho!
