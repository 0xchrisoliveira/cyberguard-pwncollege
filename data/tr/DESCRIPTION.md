Um dos principais objetivos de conectar dados com pipes é *modificá-los*.
Muitos comandos no Linux permitem transformar e tratar dados de formas extremamente práticas.
Um deles é o `tr`, que traduz (*translates*) ou substitui caracteres recebidos pela entrada padrão e os imprime na saída padrão.

Em seu uso mais elementar, o `tr` substitui o caractere informado no primeiro argumento pelo caractere correspondente informado no segundo argumento:

```console
hacker@dojo:~$ echo OWN | tr O P
PWN
hacker@dojo:~$
```

Ele também processa múltiplos caracteres simultaneamente: os caracteres nas posições do primeiro conjunto são substituídos pelos caracteres das posições equivalentes do segundo conjunto:

```console
hacker@dojo:~$ echo PWM.COLLAGE | tr MA NE
PWN.COLLEGE
hacker@dojo:~$
```

Agora é a sua vez!
Neste nível, o binário `/challenge/run` imprimirá a flag, mas com as letras maiúsculas e minúsculas invertidas (por exemplo, `A` virará `a`, e vice-versa).
Você consegue desfazer essa inversão usando `tr` e resgatar a flag?
