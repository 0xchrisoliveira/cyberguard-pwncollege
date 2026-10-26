Quando você precisa identificar alterações entre arquivos semelhantes, tentar comparar linha por linha "no olho" definitivamente não é a abordagem mais eficiente!
É aqui que o comando `diff` se torna inestimável.

O `diff` compara dois arquivos linha por linha e mostra exatamente o que é diferente entre eles.
Por exemplo:

```console
hacker@dojo:~$ cat file1
hello
world
hacker@dojo:~$ cat file2
hello
universe
hacker@dojo:~$ diff file1 file2
2c2
< world
---
> universe
```

A saída nos diz que a linha 2 foi alterada (`2c2`), com `world` no primeiro arquivo (`<`) sendo substituído por `universe` no segundo arquivo (`>`).

Às vezes, quando novas linhas são adicionadas, você verá algo como:

```console
hacker@dojo:~$ cat old
pwn
hacker@dojo:~$ cat new
pwn
college
hacker@dojo:~$ diff old new
1a2
> college
```

Isso nos informa que após a linha 1 do primeiro arquivo, o segundo arquivo possui uma linha adicional (`1a2` significa "após a linha 1 do arquivo 1, adicione a linha 2 do arquivo 2").

Agora, ao seu desafio!
Existem dois arquivos em `/challenge`:
- `/challenge/decoys_only.txt` contém 100 flags falsas (iscas)
- `/challenge/decoys_and_real.txt` contém todas as 100 flags falsas mais a única flag real

Use o comando `diff` para encontrar o que há de diferente entre esses arquivos e capture a sua flag!
