Você aprendeu como finalizar outros processos com o comando `kill`, mas às vezes você só quer se livrar do processo que está travando o seu próprio terminal!
Felizmente, os terminais contam com um atalho de teclado para isso: `Ctrl-C` (ou seja, segurar a tecla `Ctrl` e pressionar `C`) envia um sinal de "interrupção" (*interrupt*, SIGINT) para qualquer aplicativo que esteja aguardando entrada no terminal e, normalmente, isso faz com que o aplicativo seja encerrado imediatamente.

Experimente aqui!
`/challenge/run` se recusará a lhe entregar a flag até que você o interrompa.
Boa sorte!

---
Para quem quiser se aprofundar, confira este [artigo sobre terminais e "códigos de controle"](https://catern.com/posts/terminal_quirks.html) (como o `Ctrl-C`).
