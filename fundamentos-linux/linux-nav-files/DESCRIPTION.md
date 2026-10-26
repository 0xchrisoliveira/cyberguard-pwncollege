# Expedição no Sistema de Arquivos (Linux FHS & Arquivos Ocultos)

### 📖 Introdução Teórica

No ecossistema Unix e Linux, o sistema de arquivos não utiliza letras de unidades físicas (como as partições `C:` ou `D:` do Windows). Em vez disso, adota uma hierarquia lógica unificada em formato de árvore estritamente invertida, padronizada pelo **Filesystem Hierarchy Standard (FHS)**. Todo e qualquer recurso — sejam discos físicos, memórias virtuais, periféricos ou soquetes de comunicação — é mapeado a partir de um único nó-raiz denominado `/` (*root*).

```
/ (Raiz do Sistema)
├── bin -> usr/bin          [Binários essenciais do sistema: ls, cat, cp, bash]
├── etc/                    [Arquivos de configuração global: /etc/passwd, /etc/shadow]
├── home/                   [Diretórios pessoais dos usuários comuns]
├── root/                   [Diretório pessoal exclusivo do superusuário root]
├── proc/                   [Pseudofilesystem do Kernel: estado dos processos em tempo real]
├── tmp/                    [Arquivos temporários e voláteis de escrita irrestrita]
└── challenge/              [Diretório operacional do pwn.college com os desafios]
```

#### A Convenção de Arquivos Ocultos
A existência de "arquivos ocultos" no Linux não é um atributo de criptografia ou segurança profunda, mas sim uma convenção visual histórica do UNIX. Quando a função `readdir()` do kernel lê o diretório, utilitários padrão de listagem (como o comando `ls`) filtram e omitem propositalmente qualquer arquivo ou pasta cujo primeiro caractere seja um ponto (`.`, ASCII `0x2E`).

| Tipo de Arquivo | Exemplo Real | Visível com `ls`? | Visível com `ls -a`? | Impacto em Segurança |
| :--- | :--- | :---: | :---: | :--- |
| **Arquivo Comum** | `relatorio.txt` | **Sim** | Sim | Arquivos convencionais de trabalho |
| **Histórico de Shell** | `.bash_history` | **Não** | **Sim** | Comandos anteriores (frequentemente com senhas expostas) |
| **Chave de Acesso** | `.ssh/id_rsa` | **Não** | **Sim** | Chaves privadas SSH para movimentação lateral |
| **Variáveis de Ambiente** | `.env` / `.secret` | **Não** | **Sim** | Tokens de APIs, credenciais de banco de dados |

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / Pentest):** Durante a fase de pós-exploração, invasores enumeram diretórios em busca de arquivos ocultos que desenvolvedores e administradores esquecem (ex: `.git/config`, histórico de comandos `.bash_history`, credenciais de nuvem em `.aws/credentials`). Da mesma forma, atacantes instalam persistências e mineradores como arquivos ocultos em diretórios com permissão de escrita (ex: `/tmp/.xmr_miner`).
* **Segurança Defensiva (Blue Team / DFIR):** Analistas forenses e operadores de resposta a incidentes realizam varreduras com flags de listagem profunda para identificar artefatos maliciosos camuflados e auditar permissões de arquivos de configuração confidenciais.

---

### 💻 Exemplos Práticos e Sintaxe

A navegação eficiente e a auditoria de arquivos dependem do domínio dos comandos fundamentais do shell POSIX:

#### Comandos Essenciais e Flags
* `pwd` (*Print Working Directory*): Imprime o caminho absoluto do diretório atual.
* `cd <caminho>` (*Change Directory*): Altera o diretório de trabalho. Use `cd ..` para subir um nível e `cd ~` para ir à home do usuário.
* `ls`: Lista o conteúdo do diretório atual.
  * `-l`: Formato longo detalhado (permissões, proprietário, grupo, tamanho em bytes e data).
  * `-a` (*all*): Lista **todas** as entradas, incluindo arquivos ocultos (iniciados com `.`).
  * `-h` (*human-readable*): Exibe tamanhos em KB, MB ou GB em vez de bytes puros.
* `cat <arquivo>`: Concatena e exibe o conteúdo do arquivo na saída padrão.
* `find <origem> -name ".*"`: Localiza recursivamente arquivos ocultos a partir da pasta indicada.

#### Snippets de Terminal e Saídas Esperadas

1. **Listagem Padrão (Sem flag `-a`):**
   ```bash
   $ ls
   linux-nav-files  DESCRIPTION.md
   ```
   *Nota-se que apenas os arquivos visíveis são retornados.*

2. **Listagem Completa com Formato Longo (`ls -la`):**
   ```bash
   $ ls -la
   total 32
   drwxr-xr-x 2 hacker hacker 4096 Oct 26 12:00 .
   drwxr-xr-x 8 root   root   4096 Oct 26 12:00 ..
   -rw-r--r-- 1 hacker hacker   26 Oct 26 12:00 .segredo_cyberguard
   -rw-r--r-- 1 hacker hacker 2890 Oct 26 12:00 DESCRIPTION.md
   -rwsr-xr-x 1 root   root   8640 Oct 26 12:00 linux-nav-files
   ```
   *O arquivo oculto `.segredo_cyberguard` foi revelado.*

3. **Exibição do Conteúdo da Chave:**
   ```bash
   $ cat .segredo_cyberguard
   <conteudo_da_chave_secreta>
   ```

---

### 🎯 Seu Objetivo

Seu objetivo neste desafio é inspecionar o diretório `/challenge`, localizar um arquivo oculto protegido pela convenção do Unix (nomes iniciados por ponto) e extrair a chave secreta que ele contém. Em seguida, invoque o programa `/challenge/linux-nav-files` passando essa chave como argumento para comprovar que você dominou a navegação e a visualização de arquivos ocultos e obter a flag.
