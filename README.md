# 🐧 Cyberguard: Linux Luminarium (pwn.college)

Bem-vindo ao repositório oficial do **Linux Luminarium** da **Cyberguard** na plataforma **[pwn.college](https://pwn.college)**!

Este Dojo traz a trilha oficial completa do **Linux Luminarium** do pwn.college (17 módulos e 128 desafios práticos), integralmente traduzida e adaptada para **Português do Brasil (PT-BR)**. O objetivo é fornecer uma base sólida, progressiva e prática na linha de comando e no sistema operacional Linux para toda a equipe.

---

## 🗺️ Trilha Completa de Módulos (128 Desafios)

| # | Módulo | Título em Português | Desafios | Foco Principal |
|---|---|---|:---:|---|
| 1 | `hello` | Olá Hackers | 3 | Primeiro contato com a shell, prompt, comandos e argumentos. |
| 2 | `paths` | Explorando Caminhos | 8 | Navegação no sistema de arquivos, caminhos absolutos e relativos, diretório home. |
| 3 | `commands` | Compreendendo Comandos | 15 | Leitura de arquivos (`cat`), listagem (`ls`, flags ocultas), criação e remoção (`touch`, `mkdir`, `rm`). |
| 4 | `man` | Digestão de Documentação | 7 | Leitura de manuais (`man`), seções, argumentos, flags `--help` e comandos embutidos (`help`). |
| 5 | `globbing` | Globbing de Arquivos | 10 | Uso de wildcards (`*`, `?`, `[]`, `[!...]`) para expansão de arquivos no shell. |
| 6 | `piping` | Praticando Redirecionamento & Pipes | 15 | Redirecionamento de `stdin`, `stdout` e `stderr` (`>`, `>>`, `<`, `2>`, `|`, `tee`). |
| 7 | `variables` | Variáveis de Shell | 8 | Leitura, definição, exportação de variáveis de ambiente (`export`, `env`) e comandos de leitura (`read`). |
| 8 | `data` | Manipulação de Dados | 6 | Processamento de streams e filtros de texto (`head`, `tail`, `grep`, `sort`, `uniq`, `tr`, `cut`). |
| 9 | `processes` | Processos e Jobs | 10 | Listagem (`ps`), sinais (`SIGKILL`, `SIGTERM`), suspensão (`Ctrl+Z`), `fg`, `bg` e exit codes. |
| 10 | `users` | Desvendando Usuários | 4 | Troca de identidade no Linux (`su`, `sudo`), permissões e senhas de sistema. |
| 11 | `permissions` | Percebendo Permissões | 8 | Grupos (`chgrp`), proprietários (`chown`), permissões simbólicas/octais (`chmod`) e bit SUID. |
| 12 | `chaining` | Encadeando Comandos | 12 | Operadores sequenciais e condicionais (`;`, `&&`, `||`), subshells e scripts em shell. |
| 13 | `terminal-multiplexing` | Multiplexação de Terminal | 6 | Gerenciamento de sessões persistentes com `screen` e `tmux`. |
| 14 | `path` | Explorando o PATH | 5 | Variável de ambiente `PATH`, resolução de executáveis e hijacking de comandos. |
| 15 | `shenanigans` | Truques e Curiosidades | 6 | Casos especiais e comportamentos singulares do shell e do Linux. |
| 16 | `destruction` | Destruição Ousada | 5 | Gestão de recursos críticos do sistema, fork bombs e o impacto de `rm -rf /`. |
| 17 | `further-learning` | Aprendizado Contínuo | 0 | Recursos, leituras e próximos passos na jornada hacker. |

---

## 📂 Estrutura do Repositório

```bash
pwncollegeCG/
├── dojo.yml                      # Configuração do Dojo no pwn.college (17 módulos)
├── DESCRIPTION.md                # Apresentação do Dojo na plataforma pwn.college
├── README.md                     # Documentação geral deste repositório
├── common/                       # Templates Dockerfile e utilitários compartilhados
│
├── hello/                        # Módulo 1 (3 desafios)
├── paths/                        # Módulo 2 (8 desafios)
├── commands/                     # Módulo 3 (15 desafios)
├── man/                          # Módulo 4 (7 desafios)
├── globbing/                     # Módulo 5 (10 desafios)
├── piping/                       # Módulo 6 (15 desafios)
├── variables/                    # Módulo 7 (8 desafios)
├── data/                         # Módulo 8 (6 desafios)
├── processes/                    # Módulo 9 (10 desafios)
├── users/                        # Módulo 10 (4 desafios)
├── permissions/                  # Módulo 11 (8 desafios)
├── chaining/                     # Módulo 12 (12 desafios)
├── terminal-multiplexing/        # Módulo 13 (6 desafios)
├── path/                         # Módulo 14 (5 desafios)
├── shenanigans/                  # Módulo 15 (6 desafios)
├── destruction/                  # Módulo 16 (5 desafios)
├── further-learning/             # Módulo 17 (Conclusão e recursos adicionais)
│
└── Relatorios/                   # Histórico do diagnóstico da equipe (Ciclo 2026.2)
    ├── relatorio-individual-2026.2.csv
    └── DIAGNOSTICO_DESEMPENHO_CYBERGUARD.md
```

---

## 🚀 Como Treinar no Dojo

1. Acesse o Dojo oficial da Cyberguard no pwn.college:
   `https://pwn.college/dojo/cyberguard-ethical-hacking~10ef1c63`
2. Escolha o módulo desejado na lista.
3. Clique em **Start Challenge** para iniciar o container dedicado.
4. Conecte-se via terminal web (ou SSH) e siga as orientações do `DESCRIPTION.md` de cada nível.
5. Capture a flag (`/flag`) e submeta no formulário para pontuar no Scoreboard!

Bons estudos e boa jornada no terminal! 🛡️🐧
