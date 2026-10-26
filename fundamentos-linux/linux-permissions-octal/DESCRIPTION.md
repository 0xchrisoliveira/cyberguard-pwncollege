# Auditoria e Cálculo de Permissões Octais (Hardening & chmod 640)

### 📖 Introdução Teórica

O Linux adota o modelo de **Controle de Acesso Discricionário (DAC - Discretionary Access Control)** padronizado pelo POSIX. Nele, cada arquivo ou diretório possui permissões organizadas estritamente em torno de três classes de entidades:

1. **User (`u`):** O usuário proprietário do arquivo.
2. **Group (`g`):** O grupo de usuários ao qual o arquivo pertence.
3. **Others (`o`):** Todos os demais usuários existentes no sistema operacional.

Para cada uma das três classes, o sistema avalia três bits de operação fundamentais, aos quais são atribuídos pesos binários e valores octais (base 8):

* **`r` (Read / Leitura):** Peso binário `100` = **4** em octal. Permite ler o conteúdo do arquivo ou listar o diretório.
* **`w` (Write / Escrita):** Peso binário `010` = **2** em octal. Permite modificar ou truncar o arquivo ou criar/apagar arquivos no diretório.
* **`x` (Execute / Execução):** Peso binário `001` = **1** em octal. Permite carregar o arquivo na memória como processo ou navegar (`cd`) no diretório.

```
                    DÍGITO 1: USER (Dono)        = 6 (4 + 2 + 0) -> rw-
chmod 640  --->     DÍGITO 2: GROUP (Grupo)       = 4 (4 + 0 + 0) -> r--
                    DÍGITO 3: OTHER (Outros)      = 0 (0 + 0 + 0) -> ---
```

#### O Ponto Crítico: `chmod 640` vs. `chmod 755`
* **`chmod 640 arquivo` (`-rw-r-----`):** O dono pode ler e escrever (`6 = 4+2`); membros do grupo podem apenas ler (`4 = 4`); outros usuários do sistema possuem **zero permissões** (`0 = 0`). Este é o padrão ouro da indústria para proteger credenciais de banco de dados e arquivos de configuração sensíveis. Confundir `0` com acesso livre é uma das falhas mais graves em auditorias.
* **`chmod 755 arquivo` (`-rwxr-xr-x`):** O dono possui controle total de execução e escrita (`7 = 4+2+1`); o grupo e os demais usuários podem ler e executar (`5 = 4+1`), mas não modificar. Utilizado para binários públicos e scripts operacionais.

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team):** Invasores que obtêm acesso inicial com contas desprivilegiadas (como `www-data`) vasculham ativamente o sistema em busca de arquivos de configuração com permissões negligentes (como `chmod 644` ou `chmod 777`). Se um arquivo contendo senhas puder ser lido pela classe `others`, a credencial de root ou de banco é comprometida de imediato.
* **Segurança Defensiva (Blue Team / Hardening):** Guias de conformidade como CIS Benchmarks (*Center for Internet Security*) e normas ISO 27001 exigem a imposição estrita de permissões mínimas (`640` ou `600`) em arquivos confidenciais (`/etc/shadow`, `/var/log/*`, `.env`), impedindo a leitura indevida por outros processos locais.

---

### 💻 Exemplos Práticos e Sintaxe

A gestão e verificação de permissões exigem o domínio dos utilitários `chmod`, `stat` e `ls`:

#### Tabela de Equivalência Octal e Simbólica
| Notação Octal | Notação Simbólica | Dono (`u`) | Grupo (`g`) | Outros (`o`) | Aplicação Típica |
| :---: | :---: | :---: | :---: | :---: | :--- |
| **`777`** | `rwxrwxrwx` | Leitura/Escrita/Exec | Leitura/Escrita/Exec | Leitura/Escrita/Exec | **Crítico:** Nunca usar em ambientes reais |
| **`755`** | `rwxr-xr-x` | Leitura/Escrita/Exec | Leitura/Exec | Leitura/Exec | Binários e scripts públicos em `/usr/bin` |
| **`644`** | `rw-r--r--` | Leitura/Escrita | Leitura | Leitura | Arquivos públicos de texto (HTML, README) |
| **`640`** | `rw-r-----` | **Leitura/Escrita** | **Leitura** | **Nenhum** | **Arquivos confidenciais de serviços/DB** |
| **`600`** | `rw-------` | Leitura/Escrita | Nenhum | Nenhum | Chaves privadas SSH (`id_rsa`) e certificados |

#### Comandos e Flags
* `chmod <modo-octal> <arquivo>`: Aplica a máscara numérica especificada.
* `chmod <u/g/o/a><+/-/=><r/w/x> <arquivo>`: Notação simbólica (ex: `chmod u=rw,g=r,o= arquivo`).
* `stat -c "%a %A %n" <arquivo>`: Exibe a representação octal pura (`%a`), simbólica (`%A`) e o nome do arquivo (`%n`).
* `umask`: Exibe a máscara de criação de arquivos do usuário atual.

#### Snippets de Terminal e Saídas Observadas

1. **Aplicação de Permissão 640:**
   ```bash
   $ chmod 640 /tmp/teste.conf
   ```

2. **Inspeção com `ls -l`:**
   ```bash
   $ ls -l /tmp/teste.conf
   -rw-r----- 1 hacker hacker 0 Oct 26 12:00 /tmp/teste.conf
   ```
   *Note: posições 2-4 são `rw-`, posições 5-7 são `r--` e posições 8-10 são `---`.*

3. **Inspeção com `stat` (Validação Octal Exata):**
   ```bash
   $ stat -c "%a %A" /tmp/teste.conf
   640 -rw-r-----
   ```

---

### 🎯 Seu Objetivo

Existe um arquivo sensível no ambiente do desafio que atualmente possui permissões inseguras ou incorretas. Seu objetivo é auditar as permissões atuais utilizando `ls -l` e aplicar a máscara de permissões octais restritiva `640` (leitura/escrita para o proprietário, leitura para o grupo e nenhum acesso a outros usuários) utilizando o utilitário `chmod`. Em seguida, execute `/challenge/linux-permissions-octal` para que ele verifique a política de acesso e conceda a flag.
