Mesmo sem cometer erros óbvios, os usuários podem involuntariamente se colocar em risco.
Por exemplo, muitos arquivos no diretório inicial (*home*) de um usuário comum possuem _permissão de leitura para todos por padrão_ (*world-readable*), apesar de serem usados com frequência para armazenar dados confidenciais.
Acredite ou não, o seu arquivo `.bashrc` possui permissão de leitura para qualquer usuário do sistema, a menos que você altere isso explicitamente!

```console
hacker@dojo:~$ ls -l ~/.bashrc
-rw-r--r-- 1 hacker hacker 148 Jun  7 05:56 /home/hacker/.bashrc
hacker@dojo:~$
```

Você pode pensar: "Ora, pelo menos ele não é gravável por todos por padrão!".
Mas mesmo apenas com permissão de leitura, ele pode causar estragos.
Como o `.bashrc` é processado pela shell na inicialização, é nele que as pessoas costumam colocar definições de variáveis de ambiente que desejam personalizar.
Na maior parte das vezes, trata-se de itens inofensivos como `PATH`, mas às vezes as pessoas armazenam chaves de API lá para facilitar o uso diário.
Por exemplo, neste desafio:

```
zardus@dojo:~$ echo "FLAG_GETTER_API_KEY=sk-XXXYYYZZZ" > ~/.bashrc
```

Depois disso, Zardus pode facilmente fazer referência à chave de API.
Neste nível, os usuários podem usar uma chave de API válida para obter a flag:

```
zardus@dojo:~$ flag_getter --key $FLAG_GETTER_API_KEY
Correct API key! Do you want me to print the key (y/n)? y
pwn.college{HACKED}
zardus@dojo:~$
```

Naturalmente, Zardus guarda a chave dele no `.bashrc`.
Você consegue espionar e roubar essa chave para capturar a flag?

----
**NOTA:**
Quando obtiver a chave de API, basta executar `flag_getter` diretamente com o usuário `hacker`.
O `/challenge/victim` deste desafio existe apenas para compor o cenário: você não precisa utilizá-lo.
