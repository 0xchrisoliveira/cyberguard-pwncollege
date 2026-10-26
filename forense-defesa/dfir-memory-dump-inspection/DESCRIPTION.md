# Análise de Imagem de Memória RAM com Volatility

## Briefing da Missão
Após a intervenção emergencial em conformidade com a RFC 3227, o time de resposta a incidentes (CSIRT) da Cyberguard realizou a extração completa da memória física (RAM) do host atacado, gravando a imagem de memória bruta em `/tmp/cyberguard_ram_dump.raw`.

A inteligência de ameaças aponta que um malware furtivo (*fileless*) residia no espaço de endereçamento do sistema operacional, comunicando-se com um servidor de Comando e Controle (C2) e utilizando uma chave de cifragem simétrica `AES-256` gerada em tempo de execução. Como o binário nunca foi gravado no disco, a memória RAM é o único local onde essa chave existe.

Sua missão é atuar como perito forense: inspecionar o arquivo de dump de memória, localizar a chave volátil ativa e submetê-la para decodificar as comunicações adversárias.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

A análise de memória volátil é um dos pilares mais sofisticados da computação forense. No teste diagnóstico da Cyberguard, o descompasso entre teoria de disco e análise de memória foi evidente.

Quando um sistema operacional gerencia processos:
1. Cada processo recebe um espaço de memória virtual mapeado pela MMU (*Memory Management Unit*) e estruturado pelo kernel em descritores como `task_struct` (no Linux) ou `EPROCESS` (no Windows).
2. Variáveis dinâmicas, ponteiros de heap e buffers de comunicação residem nesses blocos de memória física.
3. Se o malware opera como *fileless* (usando injeção em processos legítimos via `ptrace`, reflective loading ou process hollowing), o disco rígido não contém nenhum executável suspeito.

### O Framework Volatility 3

O **Volatility 3** é o padrão pericial global para dissecar dumps de memória física sem depender do sistema operacional hospedeiro. Ele interpreta as tabelas de símbolos do kernel e permite inspecionar:
- `linux.pslist` / `windows.pslist`: Lista encadeada de processos ativos.
- `linux.malfind` / `windows.malfind`: Varredura em busca de regiões de memória com proteções atípicas (ex: memória com permissão de escrita e execução simultânea `PAGE_EXECUTE_READWRITE`).
- `linux.netscan` / `windows.netscan`: Sockets de rede abertos no instante exato do dump.
- `linux.bash` / `windows.cmdline`: Linhas de comando digitadas no terminal.

### Diagrama de Extração de Segredos em Memória

```
+-------------------------------------------------------------+
|                  IMAGEM BRUTA DA MEMÓRIA RAM                |
|                    (/tmp/cyberguard_ram_dump.raw)           |
+-------------------------------------------------------------+
|  Kernel Structures (task_struct, mm_struct, VAD Tree)      |
|  Process: rev_tunnel.elf [PID 1337]                        |
|  Stack & Heap Buffers:                                      |
|    Offset 0x00000420: "CRITICAL_VOLATILE_ARTIFACT: "       |
|    Offset 0x0000043c: "MEM_KEY_AES_256_CYBERGUARD_LIVE"    |  <-- ALVO FORENSE!
+-------------------------------------------------------------+
```

---

## Passo a Passo de Resolução

1. Execute o binário do desafio para inicializar o ambiente de simulação e ler as instruções:
   ```bash
   /challenge/dfir-memory-dump-inspection
   ```
2. Utilize o utilitário `strings` ou `grep` para vasculhar o arquivo de despejo gerado em `/tmp/cyberguard_ram_dump.raw`:
   ```bash
   strings /tmp/cyberguard_ram_dump.raw | grep MEM_KEY
   ```
   *Alternativa com grep:*
   ```bash
   grep -ao "MEM_KEY_[A-Za-z0-9_]*" /tmp/cyberguard_ram_dump.raw
   ```
3. Copie a chave identificada:
   ```text
   MEM_KEY_AES_256_CYBERGUARD_LIVE
   ```
4. Submeta a chave ao executável do desafio:
   ```bash
   /challenge/dfir-memory-dump-inspection MEM_KEY_AES_256_CYBERGUARD_LIVE
   ```
   *Ou digite-a no prompt interativo.*
5. O debriefing técnico será exibido e a flag será liberada.

---

## Dicas e Comandos de Terminal

- **Localizando Artefatos ASCII e Unicode em Binários:**
  O parâmetro `-a` força o `strings` a escanear todo o arquivo; `-t x` exibe o offset hexadecimal:
  ```bash
  strings -a -t x /tmp/cyberguard_ram_dump.raw | grep -E "(KEY|SECRET|PASSWORD)"
  ```
- **Examinando Processos Vivos Diretamente no Linux (`/proc`):**
  Se você estiver investigando uma máquina ligada, as variáveis de ambiente de qualquer processo podem ser lidas em:
  ```bash
  strings /proc/$PID/environ
  cat /proc/$PID/cmdline
  ```

---

## Aplicação Prática no Mundo Real

Em ataques de grupos APT (Advanced Persistent Threats), os operadores raramente deixam artefatos fáceis em disco. Eles utilizam frameworks como Cobalt Strike, Sliver ou Havoc, injetando *beacons* reflexivos na memória de processos de sistema. Durante a resposta a incidentes, a extração de memória é a única técnica capaz de recuperar as chaves AES e os endereços IP dos servidores de comando e controle, permitindo isolar a infraestrutura do invasor e impedir a exfiltração de gigabytes de dados sensíveis da organização.
