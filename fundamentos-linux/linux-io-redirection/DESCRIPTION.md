# Mestria em Redirecionamento (I/O POSIX e Pipelines)

### 📖 Introdução Teórica

Na filosofia do sistema operacional Unix/Linux, vigora a premissa fundamental de que "tudo é um arquivo" (*everything is a file*). Isso significa que fluxos de dados, terminais, soquetes de rede e arquivos em disco são tratados de maneira homogênea através de canais denominados **Descritores de Arquivos (File Descriptors - FDs)**.

Quando um processo é criado no Linux, ele herda automaticamente do kernel três canais padrão de entrada e saída:

| Descritor (FD) | Nome Canônico | Propósito Funcional | Dispositivo Padrão |
| :---: | :---: | :--- | :--- |
| **0** | `stdin` | Entrada padrão de dados do processo | Teclado do usuário |
| **1** | `stdout` | Saída padrão de dados normais | Monitor / Terminal |
| **2** | `stderr` | Saída padrão de mensagens de erro | Monitor / Terminal |

```
                       +-------------------------+
       stdin (FD 0)    |                         |    stdout (FD 1)
   ------------------> |   Processo em Execução  | ------------------->
   (Entrada de Dados)  |                         | (Saída Regular)
                       +-------------------------+
                                    |
                                    | stderr (FD 2)
                                    v
                           (Mensagens de Erro)
```

#### A Mecânica dos Pipes (`|`)
O operador de pipe (`|`) conecta a saída de um programa à entrada de outro sem necessidade de persistência intermediária em disco. Quando executamos `cmd1 | cmd2`, o kernel do Linux cria um buffer circular em memória (*FIFO ring buffer*, tipicamente de 64 KB). O `stdout` (FD 1) do `cmd1` é conectado diretamente à ponta de escrita do pipe, enquanto o `stdin` (FD 0) do `cmd2` é conectado à ponta de leitura. Os dois processos rodam concorrentemente.

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / Exploit Dev):** A transmissão de dados brutos e exploits automatizados depende integralmente do `stdin`. Em cenários de buffer overflow ou injeção de comandos, o invasor canaliza bytes arbitrários ou shellcodes via pipe diretamente para o processo-alvo (`python3 -c "print('A'*80)" | ./vulneravel`), contornando limitações de digitação manual.
* **Segurança Defensiva (Blue Team / SIEM):** Analistas de segurança utilizam pipelines complexos para filtrar terabytes de logs do sistema em segundos, agregando tentativas de autenticação maliciosa, isolando IPs agressores e sintetizando relatórios em tempo real sem sobrecarregar o subsistema de armazenamento.

---

### 💻 Exemplos Práticos e Sintaxe

O controle de descritores no shell Bash é exercido através de operadores especializados:

#### Tabela de Operadores de Redirecionamento
| Operador | Sintaxe | Efeito Prático |
| :---: | :--- | :--- |
| `\|` | `cmd1 \| cmd2` | Conecta o `stdout` de `cmd1` ao `stdin` de `cmd2` via buffer em memória. |
| `>` | `cmd > arquivo` | Redireciona o `stdout` para um arquivo, **sobrescrevendo** seu conteúdo anterior. |
| `>>` | `cmd >> arquivo` | Redireciona o `stdout` para um arquivo, **anexando** dados ao final (*append*). |
| `<` | `cmd < arquivo` | Alimenta o `stdin` do comando diretamente a partir do arquivo. |
| `2>` | `cmd 2> erro.log` | Redireciona exclusivamente as mensagens de erro (`stderr` / FD 2). |
| `2>&1` | `cmd > all.log 2>&1` | Faz o `stderr` seguir para o mesmo destino para o qual o `stdout` foi apontado. |

#### Snippets Práticos e Saídas do Terminal

1. **Descarte de Mensagens de Erro em Varreduras do Sistema:**
   ```bash
   $ find / -perm -4000 2>/dev/null
   ```
   *O `2>/dev/null` descarta silenciosamente centenas de mensagens de "Permissão Negada" (Permission Denied), exibindo apenas resultados autorizados.*

2. **Pipeline de Extração de Falhas de Autenticação:**
   ```bash
   $ cat /var/log/auth.log | grep "Failed password" | awk '{print $11}' | sort | uniq -c
   ```
   *Gera a contagem consolidada de tentativas falhas agrupadas por endereço IP.*

3. **Injeção de Entrada Controlada via Echo e Printf:**
   ```bash
   $ echo "DADO_TESTE" | wc -c
   11
   $ printf "DADO_TESTE" | wc -c
   10
   ```
   *Observe que o `echo` insere uma quebra de linha (`\n`), totalizando 11 bytes, enquanto o `printf` envia apenas os 10 bytes brutos.*

---

### 🎯 Seu Objetivo

O programa `/challenge/linux-io-redirection` gera um grande volume de texto na saída padrão e erros intencionais para distrair o operador. Seu objetivo é utilizar operadores de redirecionamento (`>`, `2>`, etc.) e pipelines (`|`) combinados com ferramentas de filtragem como `grep` para isolar a informação útil ou o token de autorização emitido pelo binário e capturar a flag.
