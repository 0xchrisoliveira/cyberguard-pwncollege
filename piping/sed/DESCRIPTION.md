O `grep` não é a única forma de manipular padrões. Às vezes, o dado real e o ruído inútil estão misturados dentro da mesma linha,
e queremos remover apenas esse lixo textual. Para isso, temos o `sed` (*stream editor*). O `sed` oferece uma forma prática de substituir padrões
em um fluxo de texto por outro termo. A sintaxe básica de busca e substituição é simples:

```
sed "s/oldword/newword/g"
```

`s/` - substituição (*substitute*)  
`oldword` - o termo a ser localizado e substituído  
`newword` - o substituto para `oldword` (pode ser vazio para deletar)  
`/g` - global (aplica a todas as ocorrências na linha)  

Neste desafio, `/challenge/run` imprimirá a flag, mas entre cada caractere legítimo haverá a string
"FAKEFLAG". Seu objetivo é filtrar e eliminar esse ruído da flag. Boa sorte!
