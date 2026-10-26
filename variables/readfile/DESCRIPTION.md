Frequentemente, quando usuários de shell querem carregar o conteúdo de um arquivo para uma variável, eles fazem algo como:

```console
hacker@dojo:~$ echo "test" > some_file
hacker@dojo:~$ VAR=$(cat some_file)
hacker@dojo:~$ echo $VAR
test
```

Embora funcione, isso se enquadra no que veteranos de Unix chamam de ["Uso Inútil do Cat"](https://porkmail.org/era/unix/award#cat) (*Useless Use of Cat*).
Ou seja, instanciar um processo extra inteiro apenas para ler o conteúdo de um arquivo é um desperdício desnecessário de recursos.
Acontece que você pode fazer isso usando os recursos nativos da própria shell!

Anteriormente, você usou o `read` para capturar a entrada do usuário em uma variável.
Você também aprendeu a redirecionar arquivos para a entrada de comandos usando o operador `<`.
Juntando esses dois conceitos, você pode ler arquivos diretamente na shell:

```console
hacker@dojo:~$ echo "test" > some_file
hacker@dojo:~$ read VAR < some_file
hacker@dojo:~$ echo $VAR
test
```

O que aconteceu aqui?
O exemplo redireciona `some_file` para a *entrada padrão* do `read`. Assim, quando o `read` popula a variável `VAR`, ele consome o texto diretamente do arquivo!
Agora, utilize essa técnica para carregar o arquivo `/challenge/read_me` na variável de ambiente `PWN`, e liberaremos a flag!
O conteúdo de `/challenge/read_me` muda continuamente, portanto você precisa lê-lo diretamente para a variável `PWN` em um único comando!
