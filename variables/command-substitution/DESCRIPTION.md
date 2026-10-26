No dia a dia com a shell, frequentemente você desejará armazenar a saída gerada por um comando dentro de uma variável.
Felizmente, a shell torna isso extremamente prático através do mecanismo de [_Substituição de Comandos_](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html) (*Command Substitution*)!
Observe:

```console
hacker@dojo:~$ FLAG=$(cat /flag)
hacker@dojo:~$ echo "$FLAG"
pwn.college{blahblahblah}
hacker@dojo:~$
```

Incrível, não?
Agora é a sua vez de praticar.
Capture a saída do comando `/challenge/run` diretamente em uma variável chamada `PWN`, e ela conterá a sua flag!

----
**Curiosidade:**
Você também pode encontrar o uso de crases (*backticks*) em vez de `$()`: `` FLAG=`cat /flag` `` em vez de ``FLAG=$(cat /flag)`` no exemplo acima.
Esse formato de crases é uma sintaxe legada e traz sérias desvantagens, especialmente ao aninhar substituições.
Como você aninharia comandos como `$(cat $(find / -name flag))` usando crases sem quebrar a interpretação da shell? Seria necessário escapar tudo com barras invertidas.
A recomendação oficial do pwn.college e das melhores práticas modernas de bash é sempre preferir `$(comando)` em vez de `` `comando` ``.
