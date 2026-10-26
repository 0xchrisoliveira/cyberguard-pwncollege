Neste nível, vamos praticar um pouco mais a referência a caminhos usando `.`.
Este desafio exigirá que você o execute a partir do diretório `/challenge`.
Aqui, as coisas ficam ligeiramente mais interessantes.

O Linux evita explicitamente procurar comandos no diretório atual de forma automática quando você fornece apenas o nome do executável.
Considere o seguinte exemplo:

```console
hacker@dojo:~$ cd /challenge
hacker@dojo:/challenge$ run
```

Isso *não* executará `/challenge/run`.
Na verdade, trata-se de uma medida crucial de segurança: se o Linux procurasse programas no diretório atual toda vez que você digitasse um comando simples, você poderia acidentalmente executar programas maliciosos ou inesperados no seu diretório de trabalho que tivessem o mesmo nome de utilitários centrais do sistema!
Como resultado, os comandos acima retornarão o seguinte erro:

```
bash: run: command not found
```

Exploraremos os mecanismos por trás desse conceito mais adiante, mas neste desafio aprenderemos como usar explicitamente caminhos relativos para executar `run` nesse cenário.
A forma de fazer isso é *dizer* ao Linux que você deseja explicitamente executar um programa localizado no diretório atual, utilizando o `.` como nos níveis anteriores.
Tente agora mesmo!
