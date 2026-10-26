Este módulo ensinará os conceitos MAIS fundamentais de interação com a linha de comando!
A linha de _comando_ permite que você execute _comandos_.
Quando você abre um terminal, ele executa uma "shell" de linha de comando, parecida com esta:

```console
hacker@dojo:~$
```

Isso é chamado de "prompt" (linha de solicitação), e ele está convidando você a digitar um comando.
Vamos analisar o que está acontecendo aqui:

- O `hacker` no prompt é o _nome de usuário_ (*username*) do usuário atual.
  No ambiente do Dojo do pwn.college, esse nome é sempre "hacker".
- No exemplo acima, a parte `dojo` do prompt é o _hostname_ (nome da máquina) onde a shell está rodando (esse lembrete é bastante útil se você for, por exemplo, um administrador de sistemas que gerencia vários servidores diariamente).
  No exemplo acima, o hostname é `dojo`, mas no pwn.college ele será derivado do nome do desafio que você estiver resolvendo.
- Falaremos sobre o que o `~` significa um pouco mais adiante :-)
- O `$` no final do prompt indica que o `hacker` é um usuário comum (não administrativo).
  Em módulos mais avançados do pwn.college, quando você aprender a usar exploits para obter privilégios de administrador (root), o prompt indicará isso exibindo `#` em vez de `$`, e aí você saberá que venceu!

De qualquer forma, o prompt aguarda suas ordens.
Avance para o primeiro desafio para aprender como de fato executar comandos!
