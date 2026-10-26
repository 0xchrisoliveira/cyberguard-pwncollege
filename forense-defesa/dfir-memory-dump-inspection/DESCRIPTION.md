# Análise de Imagem de Memória RAM com Volatility

### 📖 Introdução Teórica

A análise forense de memória volátil é um dos ramos mais avançados e reveladores da cibersegurança defensiva. Enquanto a análise de discos rígidos e sistemas de arquivos busca artefatos persistentes gravados no passado, a inspeção da memória RAM (*memory forensics*) revela o estado exato em que o computador se encontrava no momento preciso da captura: processos ativos em execução, fios de execução (*threads*), conexões de rede abertas, credenciais de autenticação decifradas e códigos maliciosos invisíveis no disco.

#### O Espaço de Endereçamento e Estruturas do Kernel
Em sistemas operacionais modernos, cada processo em execução recebe um espaço de memória virtual isolado, mapeado e gerenciado pelo hardware da MMU (*Memory Management Unit*):
1. **Estrutura de Processos (`task_struct` no Linux / `EPROCESS` no Windows):** O kernel mantém listas encadeadas contendo metadados de cada processo — PID, PPID, proprietário, descritores de arquivo abertos e ponteiros de memória.
2. **Árvore VAD (*Virtual Address Descriptor Tree*):** Estrutura em árvore que registra cada bloco de memória alocado para o processo e suas respectivas permissões de acesso: Leitura (`R`), Escrita (`W`) e Execução (`X`).
3. **Páginas de Memória Volátil:** Buffers dinâmicos (*heap*) e quadros de pilha (*stack*) armazenam chaves criptográficas simétricas, senhas em texto puro e dados de comunicação enquanto os programas operam.

```
+-----------------------------------------------------------------------------------------+
|                               IMAGEM BRUTA DA MEMÓRIA RAM                               |
|                              (/tmp/<token_encontrado_na_memoria>.raw)                             |
+-----------------------------------------------------------------------------------------+
|  Kernel Structures: task_struct (PID 1337, Comm: "rev_tunnel.elf")                      |
|  Active Socket:     192.168.10.45:4444 -> 10.0.0.1:8443 (Túnel C2 Ativo)               |
|  VAD Protection:    PAGE_EXECUTE_READWRITE (Alerta de Injeção Maliciosa RWX)            |
|  Process Heap/Stack:                                                                    |
|    Offset 0x00000420: "CRITICAL_VOLATILE_ARTIFACT: "                                    |
|    Offset 0x0000043c: "MEM_KEY_AES_256_CYBERGUARD_LIVE"  <--- CHAVE SIMÉTRICA DO C2!    |
+-----------------------------------------------------------------------------------------+
```

#### O Framework Volatility
O **Volatility** é o padrão pericial da indústria para análise forense de dumps de memória. O Volatility 3 não executa o binário capturado; ele interpreta de forma estática as estruturas binárias do kernel do sistema operacional a partir de tabelas de símbolos (*Intermediate Symbol Format - ISF*):
* `linux.pslist` / `windows.pslist`: Percorre a lista encadeada de processos ativos.
* `linux.pstree` / `windows.pstree`: Exibe a árvore hierárquica de relações pai-filho (`PPID -> PID`).
* `linux.malfind` / `windows.malfind`: Varre a memória em busca de regiões com proteção atípica de Leitura, Escrita e Execução simultâneas (`RWX`), sintoma clássico de injeção de malware (*Process Hollowing*, *DLL Injection*).
* `linux.netscan` / `windows.netscan`: Reconstitui os sockets de rede ativos e conexões TCP/UDP no instante da coleta.
* `linux.bash` / `windows.cmdline`: Recupera comandos digitados no terminal pelo invasor que residem nos buffers de memória.

#### Relevância Ofensiva e Defensiva
* **Segurança Defensiva (DFIR / SOC):** Permite recuperar chaves de decifração de ransomwares antes que o processo termine, identificar canais de comando e controle (C2) ocultos e isolar artefatos de intrusão que jamais tocaram o disco rígido.
* **Segurança Ofensiva (Red Team):** Compreender como os peritos extraem evidências da RAM orienta o desenvolvimento de técnicas evasivas, como ofuscação de buffers em memória e encerramento limpo de sessões.

---

### 💻 Exemplos Práticos e Sintaxe

A investigação de imagens de memória combina ferramentas especializadas (Volatility) com filtros avançados de texto e inspeção hexadecimal no terminal:

#### 1. Varredura com Volatility 3
```bash
# Listar processos em execução na imagem de memória:
vol -f /tmp/<token_encontrado_na_memoria>.raw linux.pslist

# Localizar injeções de código e regiões RWX:
vol -f /tmp/<token_encontrado_na_memoria>.raw linux.malfind

# Inspecionar sockets de rede abertos:
vol -f /tmp/<token_encontrado_na_memoria>.raw linux.netscan

# Recuperar linhas de comando digitadas:
vol -f /tmp/<token_encontrado_na_memoria>.raw linux.bash
```

#### 2. Extração Rápida de Strings e Chaves em Dumps Brutos (Raw)
Quando o perfil do kernel não está imediatamente disponível, utilitários GNU permitem localizar padrões textuais conhecidos:
```bash
# Extrair todas as cadeias de texto imprimíveis e filtrar termos de chave:
strings -a /tmp/<token_encontrado_na_memoria>.raw | grep -i MEM_KEY

# Extrair padrões com regex direta sobre o binário:
grep -ao "MEM_KEY_[A-Za-z0-9_]*" /tmp/<token_encontrado_na_memoria>.raw

# Inspecionar o offset com contexto de bytes em hexadecimal (xxd):
xxd -s 0x200 -l 128 /tmp/<token_encontrado_na_memoria>.raw
```

#### 3. Saída Esperada no Terminal
```text
[KERNEL_TASK_STRUCT pid=1337 comm=kworker_crypt]
PROCESS_NAME: rev_tunnel.elf
SOCKET_ESTABLISHED: 192.168.10.45:4444 -> 10.0.0.1:8443
ENVIRONMENT_BLOCK: USER=root PATH=/bin:/usr/bin
CRITICAL_VOLATILE_ARTIFACT: MEM_KEY_AES_256_CYBERGUARD_LIVE
VAD_PROTECTION: PAGE_EXECUTE_READWRITE
```

---

### 🎯 Seu Objetivo

Realizar a análise forense de um despejo (*dump*) de memória volátil de um sistema comprometido. Seu objetivo é investigar o estado dos processos em execução, conexões de rede e comandos executados para rastrear as ações do invasor, isolar o artefato malicioso persistido na memória e capturar a flag.
