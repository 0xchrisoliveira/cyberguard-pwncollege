# Guia 06 — Forense Digital: Ordem de Volatilidade, Memória e SIEM
**Ciclo de Capacitação 2026.2 — Cyberguard Hub & Dojo pwn.college**  
**Classificação:** Educacional / Técnico  
**Público-Alvo:** Integrantes da Cyberguard (Foco em DFIR, Blue Team e Resposta a Incidentes)

---

## 1. Introdução e o Impacto do Diagnóstico

A área de **Forense Digital e Resposta a Incidentes (DFIR)** registrou a segunda pior média de todo o teste diagnóstico da Cyberguard (apenas **12,5 pontos em 100**).

O dado mais alarmante foi que **87,5% da equipe (7 de 8 integrantes)** errou a ordem de prioridade na coleta de evidências segundo a norma internacional **RFC 3227**. A maioria dos respondentes considerou coletar logs salvos em disco ou arquivos de auditoria antes da memória RAM de um sistema comprometido.

Em um incidente de segurança corporativo real (como uma intrusão de ransomware ou invasão de rede por um APT), essa decisão equivocada causaria a **destruição definitiva e irreversível das evidências mais críticas do ataque**.

---

## 2. A Ordem de Volatilidade (RFC 3227)

Quando um analista forense ou operador de Blue Team chega a uma cena de crime cibernético ou a um servidor invadido, ele deve agir com base em um princípio basilar:

> **Colete primeiro as evidências que desaparecem mais rapidamente.**

A norma **RFC 3227** (*Guidelines for Evidence Collection and Archiving*) estabelece a ordem estrita de volatilidade que todo profissional de segurança deve memorizar:

```
MAIS VOLÁTIL (Desaparece em microssegundos/minutos)
  ▲
  │  1. Registradores da CPU e Memória Cache
  │  2. Tabelas de Roteamento, Cache ARP, Tabela de Processos, Estatísticas do Kernel
  │  3. Memória Principal (Memória RAM)
  │  4. Sistemas de Arquivos Temporários (/tmp, /dev/shm, pagefile.sys, swap)
  │  5. Discos Rígidos, SSDs e Mídias Não Voláteis
  │  6. Configurações Remotas de Logging e Dados de Tráfego de Rede
  │  7. Mídias Físicas de Arquivamento e Fitas de Backup
  ▼
MENOS VOLÁTIL (Persiste por meses ou anos)
```

### 2.1 Por que a Memória RAM DEVE Ser Coletada Antes de Qualquer Outra Coisa?

A memória RAM é volátil: **se a máquina for reiniciada ou desligada da tomada, todo o seu conteúdo é apagado para sempre**.

Enquanto os logs em disco podem esperar horas para serem clonados, a memória RAM guarda em tempo real:
1. **Malware Sem Arquivo (*Fileless Malware*) & Injeções:** Ameaças modernas operam puramente em memória (injeções DLL, *Process Hollowing*, payloads reflexivos) sem jamais tocar o disco rígido para não alertar antivírus convencionais.
2. **Conexões de Rede Ativas:** Endereços IP dos servidores de Comando e Controle (C2), portas abertas e shells reversas ativas naquele instante.
3. **Credenciais e Chaves Criptográficas em Texto Puro:** Hashes NTLM na memória do processo `lsass.exe`, senhas desprotegidas digitadas no prompt e chaves de criptografia utilizadas por ransomwares (que muitas vezes permitem recuperar os arquivos sequestrados sem pagar resgate!).
4. **Comandos Recentes:** Histórico de comandos digitados no Bash ou PowerShell que ainda não foram gravados no `.bash_history`.

> **O Erro Clássico do Diagnóstico:**  
> Acreditar que os logs em disco guardam a verdade absoluta. O invasor pode executar `rm -rf /var/log/*` ou manipular o *Event Viewer* do Windows, mas ele não consegue apagar seus rastros da memória RAM ativa enquanto o sistema estiver ligado.

---

## 3. Prática de Extração e Análise Forense de Memória

### 3.1 Coleta de Imagem de Memória (*Memory Acquisition*)

Para capturar a RAM sem alterar drasticamente o estado do sistema (*footprint* mínimo):
* **No Linux:** Utiliza-se módulos de kernel como o **LiME** (*Linux Memory Extractor*):
  ```bash
  insmod lime-$(uname -r).ko "path=/mnt/pendrive_forense/ram_dump.lime format=raw"
  ```
* **No Windows:** Utiliza-se executáveis portáteis aprovados forensicamente, como **WinPmem** ou **DumpIt**:
  ```cmd
  winpmem.exe -o C:\evidencias\memoria_servidor.raw
  ```

### 3.2 Análise com o Framework Volatility 3

Com a imagem de memória capturada, o analista utiliza o **Volatility 3** para reconstruir o que estava acontecendo na máquina:

```bash
# 1. Identificar o perfil e listar todos os processos ativos e seus pais (PPID):
vol -f memoria_servidor.raw windows.pslist

# 2. Caçar processos ocultos ou desvinculados da lista oficial (rootkits):
vol -f memoria_servidor.raw windows.psscan

# 3. Listar todas as conexões de rede abertas no momento do dump:
vol -f memoria_servidor.raw windows.netscan

# 4. Detectar injeções de código executável em memória (VAD anomalies):
vol -f memoria_servidor.raw windows.malfind

# 5. Rastrear comandos executados no prompt do terminal:
vol -f memoria_servidor.raw windows.cmdline
```

---

## 4. O Conceito e a Rigidez da Cadeia de Custódia

A **Cadeia de Custódia** (*Chain of Custody*) é o mecanismo formal, documental e cronológico que registra a trajetória completa de uma evidência digital: **desde a sua identificação e coleta, passando pelo transporte, custódia, análise laboratorial, até o seu arquivamento ou descarte**.

```
+-------------+     +---------------+     +---------------+     +---------------+
| Coleta na   | --> | Cálculo de    | --> | Análise em    | --> | Apresentação  |
| Máquina     |     | Hash SHA-256  |     | Cópia Bit a   |     | em Juízo /    |
| (Registrada)|     | (Integridade) |     | Bit (RO)      |     | Relatório     |
+-------------+     +---------------+     +---------------+     +---------------+
```

### 4.1 Requisitos Obrigatórios de um Termo de Custódia:
1. **Identificação da Evidência:** Marca, modelo, número de série, capacidade e etiquetas físicas lacradas.
2. **Pessoas Envolvidas:** Nome, cargo e assinatura de quem apreendeu, de quem transportou e de quem recebeu a evidência.
3. **Data e Hora Exatas:** Registro temporal preciso de cada movimentação.
4. **Finalidade:** O motivo específico de cada transferência de custódia.

### 4.2 Integridade Criptográfica (A Prova Pericial)
Para provar que um arquivo de evidência não foi adulterado, forjado ou corrompido durante a investigação, o perito calcula o hash criptográfico (geralmente **SHA-256**) imediatamente após a extração:

```bash
sha256sum imagem_disco_incidente.dd > hash_original.sha256
```

Se anos mais tarde, durante um processo judicial ou sindicância, o arquivo apresentar **um único bit alterado**, o hash SHA-256 resultante será completamente diferente (Efeito Avalanche), **invalidando toda a prova pericial**.

### 4.3 A Regra de Ouro da Forense Digital:
> **NUNCA realize análises forenses na mídia física ou no arquivo de imagem original!**

O procedimento correto é:
1. Extrair uma imagem forense bit-a-bit (utilizando ferramentas como `dd`, `dcfldd` ou *FTK Imager*).
2. Escrever o hash da imagem original.
3. Gerar cópias de trabalho (*working copies*).
4. Montar a imagem de trabalho em modo **estritamente somente-leitura** (*read-only*) ou com bloqueadores físicos de escrita (*write-blockers*).

---

## 5. Blue Team: O Papel de um SIEM

Na questão 12 do teste diagnóstico, a Cyberguard teve dificuldades em definir o propósito de um **SIEM** (*Security Information and Event Management*).

### O que é e o que faz um SIEM?
Em uma empresa média ou grande, centenas de servidores, firewalls, switches e endpoints geram milhões de linhas de logs por hora. Nenhum time de SOC (*Security Operations Center*) consegue ler esses logs manualmente.

O SIEM atua como o cérebro centralizador de telemetria da organização:

```
[ Firewalls ]    ──┐
[ Active Dir ]   ──┼──> [ AGENTE / SYSLOG ] ──> [ SIEM (Elastic/Splunk/Wazuh) ]
[ Servidores ]   ──┤                                    │
[ Antivírus/EDR] ──┘                                    ├──> Normalização (Parsing)
                                                        ├──> Correlação em Tempo Real
                                                        └──> Disparo de Alertas (SOC)
```

1. **Coleta e Centralização:** Agrega logs de fontes heterogêneas (Windows Event Logs, Apache, Syslog, firewall Palo Alto, autenticações de VPN).
2. **Normalização:** Converte formatos distintos para um padrão único (ex: transformar `src_ip`, `SourceAddress` e `c-ip` em um campo universal `source.ip`).
3. **Correlação em Tempo Real:** Conecta pontos aparentemente isolados aplicando regras analíticas:
   * *Exemplo de Regra:* Se o mesmo usuário registrar 15 falhas de login em menos de 1 minuto em servidores diferentes (evento 4625), seguido imediatamente por um login bem-sucedido (evento 4624) vindo de um IP estrangeiro, dispare um alerta crítico de **Força Bruta com Sucesso**.

---

## 6. O Framework MITRE ATT&CK

O **MITRE ATT&CK** (*Adversarial Tactics, Techniques, and Common Knowledge*) é uma base de conhecimento global e pública sobre o comportamento real dos grupos de invasores (APTs, cibercriminosos).

Diferente de listas de vulnerabilidades como o CVE (que listam falhas específicas de softwares), o MITRE cataloga **comportamentos e métodos**:

```
+-----------------------------------------------------------------------------------+
| TÁTICA (O Objetivo do Atacante - "POR QUÊ")                                       |
| -> Initial Access | Execution | Persistence | Privilege Escalation | Credential   |
|    Access | Lateral Movement | Collection | Exfiltration | Impact                 |
+-----------------------------------------------------------------------------------+
                                          │
                                          v
+-----------------------------------------------------------------------------------+
| TÉCNICA (O Método Utilizado - "COMO")                                             |
| -> Ex: T1003 (OS Credential Dumping: LSASS Memory)                                |
| -> Ex: T1059 (Command and Scripting Interpreter: PowerShell)                      |
| -> Ex: T1558.003 (Steal or Forge Kerberos Tickets: Kerberoasting)                 |
+-----------------------------------------------------------------------------------+
```

### Como o Blue Team Usa o MITRE ATT&CK?
* **Mapeamento de Lacunas (Gap Analysis):** O time analisa a matriz e identifica: "Se um invasor tentar despejar a memória do LSASS (T1003.001), nosso EDR ou SIEM gera algum alerta?". Se a resposta for não, o time cria a regra de detecção.
* **Testes de Red Teaming Orientados:** Os operadores ofensivos realizam simulações de adversários emulando técnicas catalogadas no MITRE para testar se os controles de defesa estão realmente operacionais.
