# O Poder do SUID (Set User ID e Escalação de Privilégios)

### 📖 Introdução Teórica

Em sistemas operacionais Unix/Linux convencionais, um processo em execução herda o identificador do usuário que o disparou, denominado **Real User ID (RUID)**. Se o usuário `hacker` (UID 1000) executa um programa, o processo roda com UID 1000 e fica terminantemente proibido pelo kernel de ler ou modificar recursos reservados ao superusuário `root` (UID 0), como `/etc/shadow` ou `/flag`.

No entanto, diversas tarefas legítimas do sistema operacional exigem privilégios elevados temporários (por exemplo, quando um usuário comum altera sua própria senha com `/usr/bin/passwd`). Para viabilizar esse comportamento sem conceder acesso irrestrito de root, o modelo POSIX introduziu o bit especial **SUID (Set User ID)**.

```
[ Usuário Comum: hacker (UID 1000) ]
              │
              │ Dispara: /usr/bin/passwd (Dono: root | Bit SUID ativo: -rwsr-xr-x)
              ▼
    ┌────────────────────────────────────────────────────────┐
    │ Processo em Execução pelo Kernel:                      │
    │   • Real UID (RUID)      = 1000 (hacker)               │
    │   • Effective UID (EUID) = 0    (root) <--- PRIVILÉGIO! │
    └────────────────────────────────────────────────────────┘
              │
              │ O Kernel autoriza a escrita pelo EUID 0:
              ▼
    [ Arquivo /etc/shadow ]
```

#### Anatomia dos Bits Especiais de Execução
Além dos bits normais de leitura, escrita e execução (`rwx`), o Linux reserva três bits especiais adicionais, representados pelo quarto dígito (mais significativo) na notação octal de 4 dígitos:

| Bit Especial | Peso Octal | Representação Simbólica | Efeito Prático no Sistema | Impacto de Segurança |
| :---: | :---: | :---: | :--- | :--- |
| **SUID** | `4000` | `-rw**s**r-xr-x` | O processo executa com o UID do **proprietário do arquivo** | Escalação de Privilégios (LPE) |
| **SGID** | `2000` | `-rwxr-**s**r-x` | O processo executa com o GID do **grupo do arquivo** | Herança forçada de grupo |
| **Sticky Bit**| `1000` | `drwxrwxrw**t**` | Apenas o criador de um arquivo pode apagá-lo na pasta | Proteção de pastas compartilhadas (`/tmp`) |

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / GTFOBins):** Quando um binário possui o bit SUID e pertence ao `root`, qualquer vulnerabilidade em seu código (como buffer overflow, injeção de comandos ou uso inadequado de argumentos de shell) permite a execução de comandos com privilégios de superusuário. Na plataforma **pwn.college**, todos os binários de desafio utilizam o bit SUID para conseguir abrir e ler o arquivo `/flag` restrito ao root.
* **Segurança Defensiva (Blue Team / Hardening):** Administradores de segurança utilizam o utilitário `find` e regras de auditoria do `auditd` para monitorar qualquer criação de novos binários SUID no sistema, já que a introdução de uma shell com SUID (`chmod 4755 /bin/bash`) é a técnica de backdoor local mais clássica existente.

---

### 💻 Exemplos Práticos e Sintaxe

A auditoria e identificação de binários SUID são realizadas através de comandos de busca de permissões:

#### Comandos e Sintaxe de Varredura
* `find / -perm -4000 -type f 2>/dev/null`: Varre todo o sistema de arquivos buscando executáveis que contenham o bit SUID (máscara octal 4000). O `2>/dev/null` descarta erros de acesso.
* `find / -perm -u=s -type f 2>/dev/null`: Sintaxe simbólica equivalente buscando o bit `s` no escopo do usuário.
* `ls -la <binario>`: Exibe os atributos do arquivo. Se o bit SUID estiver ativo, a letra `s` aparecerá no lugar do tradicional `x` do proprietário (`-rwsr-xr-x`). Se aparecer `S` maiúsculo, o SUID está ativo mas o bit de execução normal não foi concedido.
* `id`: Exibe o RUID (`uid`), o RGID (`gid`), e os IDs efetivos (`euid`/`egid`) caso difiram.

#### Snippets de Terminal e Saídas Observadas

1. **Varredura de Binários SUID Nativos no Linux:**
   ```bash
   $ find / -perm -4000 -type f 2>/dev/null
   /usr/bin/passwd
   /usr/bin/sudo
   /usr/bin/chsh
   /usr/bin/newgrp
   /challenge/linux-suid-privilege
   ```

2. **Inspeção Detalhada das Permissões do Binário SUID:**
   ```bash
   $ ls -la /challenge/linux-suid-privilege
   -rwsr-xr-x 1 root root 16384 Oct 26 12:00 /challenge/linux-suid-privilege
   ```
   *Note o `-rwsr-xr-x`: o proprietário é `root` e o bit `s` confirma a execução com privilégios elevados.*

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Para concluir a missão e extrair a flag no container pwn.college:

1. **Inspecione os atributos do executável:**
   Verifique as permissões de `/challenge/linux-suid-privilege`:
   ```bash
   ls -la /challenge/linux-suid-privilege
   ```
   *Observe o bit `s` e o usuário proprietário `root`.*

2. **Execute uma varredura para auditar outros binários SUID do sistema:**
   Pratique a localização de binários privilegiados:
   ```bash
   find /challenge -perm -4000 -type f 2>/dev/null
   ```

3. **Invoque o binário do desafio:**
   Execute o programa no terminal:
   ```bash
   /challenge/linux-suid-privilege
   ```

4. **Analise o diagnóstico de identidade emitido pelo processo:**
   O binário exibirá as identidades consultadas diretamente via chamadas de sistema `getuid()` e `geteuid()`:
   ```text
   [*] Seu Real UID: 1000 | Effective UID: 0
   [+] Executando com privilegios de root via SUID!
   pwn.college{...}
   ```
   *Constata-se que o Real UID pertence ao usuário `hacker` (1000), mas o Effective UID foi elevado pelo kernel para `root` (0), viabilizando a leitura de `/flag`.*

5. **Capture e submeta a flag:**
   Copie a sequência no padrão `pwn.college{...}` e submeta-a na plataforma.
