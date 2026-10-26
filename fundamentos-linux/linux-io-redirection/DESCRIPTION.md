# Mestria em Redirecionamento (I/O POSIX e Pipelines)

## 1. Briefing da Missão
Operador, a infraestrutura da Cyberguard utiliza streams contínuos de dados para comunicação entre processos sem dependência de escrita em disco. Um dos módulos de processamento exige a injeção do token de sincronismo operacional `CYBERGUARD_PIPELINE` diretamente através da sua entrada padrão (`stdin`).

Sua missão tática é:
1. Dominar a arquitetura de canais e descritores de arquivo POSIX (`stdin`, `stdout`, `stderr`).
2. Utilizar operadores de redirecionamento ou pipelines do Bash para canalizar o token para o binário `/challenge/linux-io-redirection`.
3. Validar a recepção correta do stream e capturar a flag.

---

## 2. Teoria Aprofundada

### 2.1 Descritores de Arquivos Padrão (File Descriptors)
No Linux, todo processo recém-criado herda da tabela do sistema três canais fundamentais de comunicação, identificados numericamente:

| Descritor | Nome | Propósito | Destino Padrão |
| :---: | :---: | :--- | :--- |
| **0** | `stdin` | Entrada padrão de dados | Teclado |
| **1** | `stdout` | Saída padrão de dados | Monitor / Terminal |
| **2** | `stderr` | Saída padrão de mensagens de erro | Monitor / Terminal |

```
                       +-------------------------+
       stdin (FD 0)    |                         |    stdout (FD 1)
   ------------------> |   Processo em Execução  | ------------------->
   (Canal de Entrada)  |                         | (Canal de Saída Normal)
                       +-------------------------+
                                    |
                                    | stderr (FD 2)
                                    v
                           (Canal de Mensagens de Erro)
```

### 2.2 Anatomia do Pipe (`|`)
Quando conectamos dois programas com o caractere pipe (`|`), o kernel do Linux cria um buffer em memória compartilhado (comumente de 64 KB). O `stdout` do comando à esquerda é mapeado diretamente para o `stdin` do comando à direita. Nenhum arquivo intermediário é gravado no disco físico!

```
[ echo "CYBERGUARD_PIPELINE" ] 
       | (FD 1 - stdout)
       v
   [ FIFO Buffer em Memória Kernel ]
       | (FD 0 - stdin)
       v
[ /challenge/linux-io-redirection ]
```

### 2.3 Relação com o Diagnóstico da Cyberguard
No relatório de desempenho, observou-se que analistas iniciantes frequentemente tentam passar dados criando arquivos manuais com `nano` ou fornecendo argumentos de linha de comando (`argv`), sem compreender que `argv` e `stdin` são interfaces de comunicação totalmente distintas. O domínio de pipes é o fundamento para automação de tarefas em massa.

---

## 3. Passo a Passo de Resolução

1. **Tentativa direta sem redirecionamento:**
   Se você executar `/challenge/linux-io-redirection` diretamente, o programa detectará que o terminal (`tty`) está interativo e exibirá a aula didática completa com instruções.

2. **Resolução Primária (Via Pipe com `echo`):**
   ```bash
   echo "CYBERGUARD_PIPELINE" | /challenge/linux-io-redirection
   ```

3. **Resolução Alternativa (Via Pipe com `printf`):**
   O `printf` garante controle exato sobre quebras de linha (`\n`):
   ```bash
   printf "CYBERGUARD_PIPELINE\n" | /challenge/linux-io-redirection
   ```

4. **Resolução com Redirecionamento de Arquivo (`<`):**
   ```bash
   echo "CYBERGUARD_PIPELINE" > /tmp/token.txt
   /challenge/linux-io-redirection < /tmp/token.txt
   ```

---

## 4. Dicas e Comandos de Terminal

| Operador | Exemplo Prático | Descrição |
| :---: | :--- | :--- |
| `\|` | `cat logs.txt \| grep "ALERT"` | Conecta o stdout do comando 1 ao stdin do comando 2. |
| `>` | `nmap 10.10.10.5 > scan.txt` | Sobrescreve o arquivo com a saída normal (stdout). |
| `>>` | `echo "token" >> secrets.env` | Anexa novos dados ao final do arquivo sem sobrescrever. |
| `<` | `mysql -u root < dump.sql` | Alimenta o programa diretamente a partir de um arquivo. |
| `2>` | `find / -perm -4000 2>/dev/null` | Descarta mensagens de erro (FD 2) enviando para `/dev/null`. |
| `2>&1` | `cmd > all.log 2>&1` | Redireciona stderr para o mesmo destino de stdout. |

---

## 5. Aplicação Prática no Mundo Real

* **Construção de Filtros de SIEM e Log Hunting:** Analistas de Blue Team usam pipelines para filtrar gigabytes de logs em segundos:
  ```bash
  journalctl -u ssh | grep "Failed password" | awk '{print $11}' | sort | uniq -c | sort -nr
  ```
* **Armamento de Exploits em Red Team:** Em testes de invasão e CTFs, buffers binários e payloads de shellcode gerados via Python são transmitidos diretamente para o binário vulnerável via pipe:
  ```bash
  python3 -c "print('A'*72 + '\xef\xbe\xad\xde')" | /vuln/binary
  ```
