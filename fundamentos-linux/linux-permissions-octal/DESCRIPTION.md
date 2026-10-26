# Auditoria e Cálculo de Permissões Octais (Hardening & chmod 640)

## 1. Briefing da Missão
Operador, a equipe de Blue Team da Cyberguard identificou uma não conformidade grave durante a auditoria de um servidor de aplicação. O arquivo de configuração confidencial `/tmp/config.conf` precisa ser protegido com a política restritiva de acesso mais celebrada da indústria: **permissão octal 640**.

Sua missão tática é:
1. Analisar a decomposição matemática da notação octal de permissões no Linux.
2. Criar ou ajustar o arquivo `/tmp/config.conf` para que contenha exatamente as permissões `640` (`-rw-r-----`).
3. Executar o auditor `/challenge/linux-permissions-octal` para validar a conformidade e resgatar a flag.

---

## 2. Teoria Aprofundada

### 2.1 A Tríade de Classes e Pesos Octais
No modelo de controle de acesso discricionário (DAC - *Discretionary Access Control*) do padrão POSIX, cada recurso pertence a um **Usuário Proprietário (`u`)** e a um **Grupo Proprietário (`g`)**, com regras para os **Demais Usuários (`o`)**.

Cada classe recebe um conjunto de três bits correspondentes a:
* **`r` (Read / Leitura):** Peso binário `100` = **4** em octal.
* **`w` (Write / Escrita):** Peso binário `010` = **2** em octal.
* **`x` (Execute / Execução):** Peso binário `001` = **1** em octal.

```
                    DÍGITO 1: USER (Dono)        = 6 (4 + 2 + 0) -> rw-
chmod 640  --->     DÍGITO 2: GROUP (Grupo)       = 4 (4 + 0 + 0) -> r--
                    DÍGITO 3: OTHER (Outros)      = 0 (0 + 0 + 0) -> ---
```

### 2.2 Tabela de Comparação de Padrões Operacionais

| Notação Octal | Notação Simbólica | Dono (`u`) | Grupo (`g`) | Outros (`o`) | Cenário de Uso Recomendado |
| :---: | :---: | :---: | :---: | :---: | :--- |
| **`777`** | `rwxrwxrwx` | Leitura/Escrita/Exec | Leitura/Escrita/Exec | Leitura/Escrita/Exec | **INACEITÁVEL EM PRODUÇÃO** (vulnerabilidade crítica) |
| **`755`** | `rwxr-xr-x` | Leitura/Escrita/Exec | Leitura/Exec | Leitura/Exec | Binários públicos e scripts executáveis |
| **`644`** | `rw-r--r--` | Leitura/Escrita | Leitura | Leitura | Arquivos públicos não executáveis (HTML, CSS) |
| **`640`** | `rw-r-----` | **Leitura/Escrita** | **Leitura** | **NENHUM** | **Padrão Ouro:** Arquivos de credenciais de serviços e DBs |
| **`600`** | `rw-------` | Leitura/Escrita | NENHUM | NENHUM | Chaves privadas SSH (`id_rsa`) e certificados sensíveis |

### 2.3 Raio-X do Diagnóstico da Cyberguard (Questão Q01)
Na avaliação diagnóstica inicial do Ciclo 2026.2, **62,5% dos membros avaliados falharam** ao responder o efeito prático do comando `chmod 640`. O equívoco predominante foi presumir que permissões com o dígito zero concederiam acesso herdado da pasta ou leitura pública. 

O dígito `0` significa bloqueio absoluto: **nenhum bit concedido**.

---

## 3. Passo a Passo de Resolução

1. **Tentativa inicial (Diagnóstico prévio):**
   ```bash
   /challenge/linux-permissions-octal
   ```
   *Se o arquivo não existir, o binário alertará e exibirá a aula teórica completa.*

2. **Criação do arquivo alvo no diretório `/tmp`:**
   ```bash
   touch /tmp/config.conf
   ```

3. **Aplicação rigorosa da máscara octal 640:**
   ```bash
   chmod 640 /tmp/config.conf
   ```

4. **Inspeção forense da máscara aplicada:**
   ```bash
   ls -l /tmp/config.conf
   ```
   *Saída esperada:*
   ```text
   -rw-r----- 1 hacker hacker 0 Oct 26 12:00 /tmp/config.conf
   ```
   *Ou inspecionando em formato octal puro com `stat`:*
   ```bash
   stat -c "%a %A %n" /tmp/config.conf
   # Deve retornar: 640 -rw-r----- /tmp/config.conf
   ```

5. **Execução do auditor para resgate da flag:**
   ```bash
   /challenge/linux-permissions-octal
   ```

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição |
| :--- | :--- |
| `chmod <octal> <arquivo>` | Define permissões pelo método numérico octal (ex: `chmod 640 app.conf`). |
| `chmod u=rw,g=r,o= <arquivo>` | Sintaxe simbólica equivalente a `640`. |
| `stat -c "%a %A" <arquivo>` | Exibe a representação octal e simbólica do arquivo de forma limpa. |
| `umask` | Exibe a máscara de criação padrão que subtrai bits de novos arquivos. |

---

## 5. Aplicação Prática no Mundo Real

* **Proteção de Segredos em Servidores Web:** Em arquiteturas Apache ou Nginx com PHP/Python, arquivos como `wp-config.php` ou `.env` contêm senhas de banco de dados. Deixar esses arquivos com `644` permite que outros usuários locais no mesmo servidor compartilhado leiam as senhas. Configurar `640` com dono `root` e grupo `www-data` restringe o acesso estritamente ao processo do servidor web.
* **Hardening CIS Benchmarks:** Todos os guias de conformidade do *Center for Internet Security* (CIS) exigem permissões `640` ou `600` para arquivos de logs em `/var/log` e chaves privadas em `/etc/ssl/private`.
