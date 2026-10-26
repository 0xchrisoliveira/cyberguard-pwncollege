# Expedição no Sistema de Arquivos (Linux FHS & Arquivos Ocultos)

## 1. Briefing da Missão
Operador, a equipe de inteligência da Cyberguard interceptou um container Linux de testes operacionais. Durante o procedimento de auditoria rápida, um artefato confidencial foi deliberadamente camuflado sob a hierarquia de arquivos do diretório `/challenge`.

Sua missão tática consiste em:
1. Inspecionar o diretório operacional utilizando ferramentas de listagem profunda.
2. Identificar arquivos ocultos protegidos pela convenção de ponto do UNIX.
3. Extrair a chave secreta do arquivo oculto.
4. Fornecer a chave como argumento de validação para o binário `/challenge/linux-nav-files` e resgatar a flag do sistema.

---

## 2. Teoria Aprofundada

### 2.1 A Estrutura FHS (Filesystem Hierarchy Standard)
No ecossistema Linux, o sistema de arquivos não utiliza letras de unidade (como `C:` ou `D:`). Toda a estrutura deriva de um único nó-raiz (`/`):

```
/ (Raiz do Sistema)
├── bin -> usr/bin          [Binários essenciais do sistema: ls, cat, cp]
├── etc/                    [Configurações globais e senhas do sistema: /etc/passwd]
├── home/                   [Diretórios pessoais dos usuários convencionais]
├── root/                   [Diretório home do superusuário root]
├── proc/                   [Pseudofilesystem do Kernel: estado dos processos em tempo real]
├── tmp/                    [Arquivos temporários e voláteis de escrita irrestrita]
└── challenge/              [Diretório da missão pwn.college com os binários de desafio]
```

### 2.2 O Fenômeno dos Arquivos Ocultos no Unix
A convenção de ocultação de arquivos no Linux remonta aos primórdios do UNIX. Trata-se de uma regra de visualização: a chamada de sistema `readdir()` e o utilitário `ls` simplesmente filtram e omitem qualquer nome de entrada cujo primeiro byte seja o caractere ASCII ponto (`.`, hexadecimal `0x2E`).

| Tipo de Arquivo | Exemplo | Visível com `ls` Simples? | Visível com `ls -a`? | Risco em Auditorias |
| :--- | :--- | :---: | :---: | :--- |
| **Arquivo Padrão** | `relatorio.txt` | Sim | Sim | Exposição direta |
| **Arquivo Oculto de Shell** | `.bash_history` | **Não** | **Sim** | Histórico de comandos com senhas |
| **Arquivo Oculto de Chave** | `.ssh/id_rsa` | **Não** | **Sim** | Chaves privadas SSH |
| **Arquivo Oculto de Configuração** | `.env` / `.secret` | **Não** | **Sim** | Tokens de API e credenciais de DB |

### 2.3 Relação com o Diagnóstico da Cyberguard
No relatório diagnóstico da equipe (Ciclo 2026.2), operadores em início de jornada apresentaram dificuldades recorrentes em reconhecer que arquivos ocultos não estão criptografados nem inacessíveis; estão apenas invisíveis à listagem padrão. O comando `ls -la` é a primeira ferramenta de sobrevivência de qualquer auditor ou analista forense.

---

## 3. Passo a Passo de Resolução

1. **Acesse o ambiente do desafio:**
   Conecte-se ao terminal do pwn.college e navegue até a pasta do desafio:
   ```bash
   cd /challenge
   pwd
   ```

2. **Execute a listagem padrão (observe o comportamento):**
   ```bash
   ls
   ```
   *Resultado esperado:* Apenas o binário `linux-nav-files` e este arquivo `DESCRIPTION.md` aparecerão. O arquivo secreto não é exibido.

3. **Execute a listagem exaustiva com parâmetros `-la`:**
   ```bash
   ls -la
   ```
   *Resultado esperado:* Todas as entradas serão reveladas, incluindo `.segredo_cyberguard`.

4. **Inspecione o conteúdo do arquivo oculto:**
   ```bash
   cat .segredo_cyberguard
   ```
   *Você visualizará a chave secreta:* `cyberguard_linux_fhs_2026`.

5. **Execute o validador passando a chave como parâmetro:**
   ```bash
   /challenge/linux-nav-files cyberguard_linux_fhs_2026
   ```
   *Ou em uma única linha elegante:*
   ```bash
   /challenge/linux-nav-files $(cat /challenge/.segredo_cyberguard)
   ```

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição Operacional |
| :--- | :--- |
| `pwd` | Imprime o diretório de trabalho atual (*Print Working Directory*). |
| `ls -la` | Lista todos os arquivos com atributos detalhados (permissões, dono, tamanho). |
| `cat <arquivo>` | Concatena e exibe o conteúdo de um arquivo de texto no terminal. |
| `head -n 5 <arquivo>` | Exibe as 5 primeiras linhas de um arquivo grande. |
| `find . -name ".*"` | Busca recursivamente todos os arquivos ocultos a partir do diretório atual. |

---

## 5. Aplicação Prática no Mundo Real

* **Incident Response & Blue Team:** Atores de ameaça frequentemente salvam ferramentas de movimentação lateral e scripts de persistência em arquivos ocultos (ex: `/tmp/.xmr_miner`, `/var/tmp/.bd`). Analistas devem sempre incluir `-a` em rotinas de triage.
* **Red Team & Pentest:** A enumeração de diretórios de usuário com `ls -la ~` frequentemente descobre arquivos `.bash_history` repletos de credenciais inseridas por engano por administradores, ou arquivos `.aws/credentials` com tokens de nuvem em texto plano.
