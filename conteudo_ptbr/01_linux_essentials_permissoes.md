# Guia 01 — Fundamentos de Linux, Permissões e Auditoria Operacional
**Ciclo de Capacitação 2026.2 — Cyberguard Hub & Dojo pwn.college**  
**Classificação:** Educacional / Técnico  
**Público-Alvo:** Integrantes da Cyberguard (Ênfase nas lacunas de Fundamentos Operacionais e Hardening)

---

## 1. Introdução e Contexto do Diagnóstico

Na avaliação diagnóstica de conhecimentos prévios da Cyberguard (Ciclo 2026.2), **62,5% da equipe (5 de 8 membros)** falhou na interpretação e no cálculo de permissões octais no Linux — especificamente na questão sobre `chmod 640`. 

Para atuar com eficácia em **Segurança Ofensiva (Red Team / CTFs)** ou em **Segurança Defensiva (Blue Team / Hardening)**, o domínio da árvore de arquivos e do modelo de segurança DAC (*Discretionary Access Control*) do Linux é um pré-requisito elementar. Sem entender quem pode ler, escrever ou executar um binário ou arquivo de configuração, torna-se impossível realizar auditorias de privilégio, escalar privilégios (*privilege escalation*) ou proteger ativos corporativos.

---

## 2. A Hierarquia do Sistema de Arquivos Linux (FHS)

No Linux, diferentemente do Windows, não existem unidades como `C:` ou `D:`. Tudo está contido sob a raiz única representada pela barra `/`. Além disso, vigora o axioma fundamental do UNIX: **"Quase tudo é um arquivo"** (arquivos de texto, diretórios, conexões de rede e até mesmo dispositivos de hardware).

```
/ (Raiz do Sistema)
├── bin -> usr/bin          (Binários essenciais do sistema e comandos básicos)
├── sbin -> usr/sbin        (Binários administrativos executados prioritariamente pelo root)
├── etc/                    (Arquivos de configuração do sistema e de serviços)
├── home/                   (Diretórios dos usuários convencionais: /home/alice, /home/bob)
├── root/                   (Diretório 'home' do superusuário root)
├── tmp/                    (Arquivos temporários, comumente com Sticky Bit habilitado)
├── var/                    (Dados variáveis: logs em /var/log, spool, caches)
├── proc/                   (Pseudofilesystem do Kernel: estado dos processos em memória)
├── dev/                    (Nós de dispositivos: discos /dev/sda, interfaces seriais, /dev/null)
└── opt/                    (Softwares e ferramentas de terceiros adicionais)
```

### Principais Diretórios de Interesse em Auditoria e Pentest:

1. **/etc/**: O coração das configurações do Linux. Contém arquivos sensíveis como:
   * `/etc/passwd`: Lista de usuários do sistema (legível por todos).
   * `/etc/shadow`: Hashes das senhas dos usuários (restrito exclusivamente ao `root`).
   * `/etc/sudoers`: Regras de privilégios e elevação com `sudo`.
   * `/etc/crontab` e `/etc/cron.*`: Tarefas agendadas periódicas.
2. **/proc/**: Não existe no disco físico; é uma interface de comunicação direta em tempo real com o Kernel do Linux.
   * `/proc/[PID]/cmdline`: Linha de comando com a qual o processo PID foi iniciado.
   * `/proc/[PID]/environ`: Variáveis de ambiente daquele processo (frequentemente vazam tokens e senhas).
   * `/proc/[PID]/maps`: Mapa de memória do processo (crucial em engenharia reversa e exploits).
3. **/dev/**: Arquivos especiais de dispositivos.
   * `/dev/null`: Descarta qualquer dado enviado para ele.
   * `/dev/urandom`: Gerador de bytes pseudoaleatórios criptograficamente seguros do kernel.
   * `/dev/shm`: Memória compartilhada montada como tmpfs (RAM pura, rápida e discreta para invasores).
4. **/tmp/ e /var/tmp/**: Diretórios de escrita pública utilizados para despejo temporário de ferramentas, scripts e payloads em pós-exploração.

---

## 3. Mecânica de Permissões no Linux: Octal vs Simbólico

Todo arquivo ou diretório no Linux possui três níveis de permissão associados a três categorias de usuários:

| Categoria | Sigla | Descrição |
| :--- | :---: | :--- |
| **User (Dono)** | `u` | O proprietário original do arquivo ou diretório |
| **Group (Grupo)** | `g` | Os membros do grupo associado ao arquivo |
| **Others (Outros)** | `o` | Todos os demais usuários existentes no sistema |

### 3.1 A Tríade Básica de Permissões

Cada classe pode ter até três permissões fundamentais:
* **Read (Leitura - `r`):** Valor binário `100` = **4** em octal. Permite abrir e ler o arquivo ou listar o conteúdo de um diretório.
* **Write (Escrita - `w`):** Valor binário `010` = **2** em octal. Permite modificar/salvar o arquivo ou criar/apagar itens dentro de um diretório.
* **Execute (Execução - `x`):** Valor binário `001` = **1** em octal. Permite rodar o arquivo como programa/script ou entrar (`cd`) em um diretório.

A soma dos valores octais define o número correspondente:
* `4 + 2 + 1 = 7` -> `rwx` (Leitura, Escrita e Execução)
* `4 + 2 + 0 = 6` -> `rw-` (Leitura e Escrita)
* `4 + 0 + 1 = 5` -> `r-x` (Leitura e Execução)
* `4 + 0 + 0 = 4` -> `r--` (Apenas Leitura)
* `0 + 0 + 0 = 0` -> `---` (Nenhum Acesso)

---

## 4. Dissecando a Questão do Teste: `chmod 640`

Na pergunta diagnóstica que gerou 62,5% de erro na Cyberguard, foi questionado o efeito prático do comando:
```bash
chmod 640 database_config.env
```

### Decomposição Matemática do 640:
```
 Dígito 1 (Dono / User)     = 6  ->  4 + 2 + 0  ->  rw-  (Leitura e Escrita)
 Dígito 2 (Grupo / Group)   = 4  ->  4 + 0 + 0  ->  r--  (Apenas Leitura)
 Dígito 3 (Outros / Others) = 0  ->  0 + 0 + 0  ->  ---  (Nenhum Acesso)
```

No formato simbólico do `ls -la`, o arquivo `database_config.env` exibirá a seguinte máscara de bits:
```
-rw-r----- 1 root webadmins 1024 Oct 26 10:00 database_config.env
```

### Por que `640` é um Padrão da Indústria em Segurança Defensiva?
Considere uma aplicação web rodando sob o usuário `www-data` e mantida por um grupo de administradores chamado `webadmins`:
1. **O Dono (`root`)** pode editar e gerenciar o arquivo com segurança (`rw-`).
2. **O Grupo da Aplicação (`webadmins` ou `www-data`)** precisa apenas consultar as credenciais de banco de dados para estabelecer a conexão, mas nunca deve ter permissão de alterar esse arquivo (`r--`).
3. **Outros usuários locais (`others`)**, como estagiários, contas de serviços de telemetria ou invasores sem privilégio de grupo, **não conseguem sequer ler** uma única linha do arquivo (`---`).

Se esse arquivo estivesse com `644` (`-rw-r--r--`), qualquer usuário local comum poderia executar `cat database_config.env` e obter senhas em texto puro!

---

## 5. Permissões Especiais: SUID, SGID e Sticky Bit

Além dos bits básicos, o Linux implementa três permissões especiais cruciais para segurança ofensiva e defensiva. Elas ocupam o quarto dígito mais à esquerda no formato octal (`[Especial][User][Group][Other]`).

```
  +------------------+------------------+------------------+
  |    SUID (4xxx)   |    SGID (2xxx)   | Sticky Bit (1xxx)|
  +------------------+------------------+------------------+
  | Executa como o   | Executa com o    | Somente o dono   |
  | DONO do binário  | GRUPO do arquivo | do arquivo pode  |
  | (ex: root)       | ou herda grupo   | apagá-lo da pasta|
  +------------------+------------------+------------------+
```

### 5.1 SUID (Set User ID — Valor Octal: 4)
* **Comando:** `chmod 4755 /usr/bin/exemplo` ou `chmod u+s /usr/bin/exemplo`
* **Máscara:** `-rwsr-xr-x`
* **Comportamento:** O processo é executado com os privilégios do **proprietário** do binário, não do usuário que o invocou.
* **Exemplo Legítimo:** O utilitário `/usr/bin/passwd`. Para alterar a própria senha, um usuário comum precisa escrever no arquivo restrito `/etc/shadow`. O binário `passwd` tem o bit SUID com dono `root`, permitindo a escrita controlada.
* **Vetor Ofensivo (CTFs e Red Team):** Se um binário como `find`, `bash`, `vim`, ou `nmap` possuir o bit SUID como root, qualquer invasor comum pode usá-lo para obter um terminal interativo com privilégios de superusuário (consulte o portal [GTFOBins](https://gtfobins.github.io/)).

### 5.2 SGID (Set Group ID — Valor Octal: 2)
* **Comando:** `chmod 2755 /diretorio` ou `chmod g+s /diretorio`
* **Máscara:** `-rwxr-sr-x` ou `drwxr-sr-x`
* **Comportamento:** 
  * Em arquivos: Executa com os privilégios do **grupo** do binário.
  * Em diretórios: Qualquer arquivo ou subpasta criado dentro desse diretório herdará automaticamente o **grupo** do diretório pai, e não o grupo primário de quem o criou.

### 5.3 Sticky Bit (Valor Octal: 1)
* **Comando:** `chmod 1777 /tmp` ou `chmod +t /tmp`
* **Máscara:** `drwxrwxrwt`
* **Comportamento:** Aplicado a diretórios de escrita global (onde qualquer um pode criar arquivos). Garante que **somente o proprietário do arquivo (ou o root)** possa excluí-lo ou renomeá-lo, impedindo que usuários maliciosos apaguem arquivos alheios.

---

## 6. Comandos Fundamentais para Auditoria e CTF

Para caçar vulnerabilidades no sistema, o analista deve dominar o ecossistema de utilitários de linha de comando:

### 6.1 Caça a Binários SUID com `find`
Para listar todos os binários SUID com dono root ignorando erros de permissão negada:
```bash
find / -perm -4000 -user root -type f -exec ls -la {} + 2>/dev/null
```
* `-perm -4000`: Filtra arquivos que contenham o bit SUID setado.
* `-type f`: Apenas arquivos regulares.
* `2>/dev/null`: Descarta mensagens de erro (stderr) enviando-as ao buraco negro.

### 6.2 Busca e Extração com `grep` e `ripgrep`
Busca recursiva por chaves de API, senhas ou tokens em arquivos de texto:
```bash
grep -rnw '/var/www/' -e "DB_PASSWORD" --exclude="*.log" 2>/dev/null
```

### 6.3 Processamento Rápido com `awk` e `sed`
Extrair apenas os usuários com shell de login válido `/bin/bash` a partir de `/etc/passwd`:
```bash
awk -F: '$7 == "/bin/bash" {print $1, $3}' /etc/passwd
```
Substituir uma configuração de segurança em massa sem abrir editor manual:
```bash
sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
```

### 6.4 Redirecionamentos e Pipes
* `comando > arquivo`: Sobrescreve a saída padrão (stdout, FD 1).
* `comando >> arquivo`: Concatena a saída padrão no final do arquivo.
* `comando 2> erro.log`: Redireciona mensagens de erro (stderr, FD 2).
* `comando &> tudo.log`: Redireciona simultaneamente stdout e stderr.
* `comando 2>&1`: Redireciona o canal de erro para onde o canal padrão estiver apontando.
* `comando1 | comando2`: Envia a saída padrão de 1 diretamente como entrada padrão de 2.

### 6.5 Variáveis de Ambiente e Perigos de PATH Hijacking
Quando um binário SUID executa um comando interno sem especificar o caminho absoluto (ex: chama `system("service apache2 restart")` em vez de `/usr/sbin/service`), ele confia na variável de ambiente `$PATH`:
```bash
# Injetando um diretório malicioso no topo do PATH:
export PATH=/tmp/fake_bin:$PATH

# Criando um payload com o mesmo nome do utilitário esperado:
echo -e '#!/bin/bash\n/bin/bash -p' > /tmp/fake_bin/service
chmod +x /tmp/fake_bin/service

# Ao executar o binário SUID vulnerável, ele chamará nosso script falso como root!
```

---

## 7. Exercício Prático de Fixação

1. Calcule a representação octal para um arquivo onde o dono possui Leitura e Execução, o grupo possui Apenas Leitura e outros não possuem nenhum acesso.  
   *Resposta: `540` (`r-xr-----`).*
2. Por que o comando `chmod 777` em ambientes corporativos ou de produção é considerado uma má prática gravíssima?  
   *Resposta: Porque concede permissão irrestrita de leitura, alteração e execução para qualquer processo ou usuário local, violando o princípio do menor privilégio.*
