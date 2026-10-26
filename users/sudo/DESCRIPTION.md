Antigamente, um sistema Linux típico possuía uma senha para o `root`, que os administradores utilizavam com `su` para se tornarem `root` (após se autenticarem com a senha da sua conta de usuário comum).
Porém, senhas de `root` são trabalhosas de gerenciar, elas (ou seus hashes!) podem vazar e não escalam bem em ambientes corporativos ou grandes parques de servidores.
Para resolver isso, nas últimas décadas, o ecossistema migrou da administração via `su` para a administração via `sudo` (*Curiosidade*: `sudo` originalmente significava **su**peruser **do**, mas com o tempo passou a ser interpretado como "`su` 'do'", e como `su` significa "substitute user", o significado moderno de `sudo` é "substitute user, do" — substituir usuário e executar).

Diferente do `su`, que por padrão abre uma nova shell como o usuário especificado, o `sudo` executa por padrão um comando específico diretamente como `root`:

```console
hacker@dojo:~$ whoami
hacker
hacker@dojo:~$ sudo whoami
root
hacker@dojo:~$
```

Ou, em um exemplo mais relevante para capturar flags:

```console
hacker@dojo:~$ grep hacker /etc/shadow
grep: /etc/shadow: Permission denied
hacker@dojo:~$ sudo grep hacker /etc/shadow
hacker:$6$Xro.e7qB3Q2Jl2sA$j6xffIgWn9xIxWUeFzvwPf.nOH2NTWNJCU5XVkPuONjIC7jL467SR4bXjpVJx4b/bkbl7kyhNquWtkNlulFoy.:19921:0:99999:7:::
hacker@dojo:~$
```

Diferente do `su`, que depende da senha da conta de destino (`root`), o `sudo` consulta políticas configuradas para determinar se o usuário atual tem autorização para executar comandos como `root` (geralmente exigindo apenas a senha do próprio usuário ou nenhuma senha).
Essas políticas são definidas no arquivo `/etc/sudoers` e, embora estejam fora do escopo deste nível, há excelentes [recursos](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file) para aprender mais sobre isso!

Portanto, o mundo adotou o `sudo` e deixou o `su` para trás na administração de sistemas.
Inclusive, o Modo Privilegiado (*Privileged Mode*) do próprio pwn.college funciona concedendo acesso a você via `sudo` para elevar privilégios!

Neste nível, concederemos acesso a você via `sudo`, e você usará isso para ler a flag.
Simples e direto!

----
**NOTA:**
Após este nível, habilitaremos o Modo Privilegiado (*Privileged Mode*)!
No ambiente de trabalho, clique no cadeado fechado para reiniciar o desafio em Modo Privilegiado e clique no cadeado aberto para retornar ao modo normal.
O hostname no prompt do terminal termina com `~practice` enquanto o Modo Privilegiado estiver ativo.
O Modo Privilegiado dá a você acesso total ao `sudo` para inspecionar e depurar o ambiente, mas fornece uma flag de teste (*placeholder*) que não pode ser submetida para pontuação.
