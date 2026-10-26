# RELATÓRIO EXECUTIVO & DIAGNÓSTICO ANALÍTICO DE DESEMPENHO
## Ciclo 2026.2 — Cyberguard Hub & Dojo pwn.college
**Data de Emissão:** 26 de Outubro de 2026  
**Classificação:** Confidencial / Uso Interno  
**Autoria:** Divisão de Segurança Ofensiva & Defensiva — Cyberguard  
**Fonte de Dados:** `relatorio-individual-2026.2.csv` (8 Membros Avaliados / 20 Indicadores Técnicos)

---

## 1. Sumário Executivo

O Teste de Conhecimentos Prévios do ciclo **2026.2** avaliou 8 integrantes da **Cyberguard** através de um instrumento diagnóstico composto por 20 questões objetivas distribuídas em 12 domínios de competência em segurança da informação, cobrindo vertentes de Red Team, Blue Team, DevSec e Fundamentos de Computação.

O diagnóstico consolidado revelou uma equipe com **assimetria severa de competências** e um desvio padrão acentuado entre os integrantes:
* **Média Geral de Acertos:** **39,38%** (7,88 de 20 questões).
* **Mediana da Equipe:** **32,50%** (6,50 de 20 questões).
* **Taxa de Polarização:** 37,5% da equipe (3 membros) responde por **71,4%** de todos os acertos computados, enquanto 50,0% da equipe (4 membros) obteve aproveitamento igual ou inferior a 20%.

```
Distribuição de Performance (Aproveitamento em %):
[0% - 20%]   ████ (4 membros) -> Luiz Paulo (5%), Ismael (10%), Matheus (10%), Joao Vitor (20%)
[21% - 50%]  █    (1 membro)  -> Lauren (45%)
[51% - 70%]  ██   (2 membros) -> Silvio (70%), Arthur (70%)
[71% - 100%] █    (1 membro)  -> Giovanni (85%)
```

### Principais Conclusões Estratégicas
1. **O Paradoxo Binário (Abismo Baixo Nível):** A totalidade dos respondentes (8/8) acertou a periculosidade conceitual de `strcpy` em C. Entretanto, **100% dos membros erraram a função do registrador EIP/RIP** em x86/x64 e **87,5% erraram a distinção entre análise estática e dinâmica**. Isso diagnostica um conhecimento puramente "livresco" de código seguro desprovido de qualquer entendimento prático do modelo de execução na CPU, pilha (*stack frame*) ou corrupção de memória.
2. **Defasagem nos Fundamentos Operacionais:** Erros alarmantes em tópicos que deveriam ser pré-requisitos elementares: **62,5% erraram o cálculo de permissões octais no Linux (`chmod 640`)** e **62,5% erraram a ordem clássica do handshake TCP (SYN, SYN-ACK, ACK)**. Sem esses alicerces, o aprendizado de exploração avançada e hardening corporativo fica inviabilizado.
3. **Cegueira Forense e de Resposta a Incidentes (DFIR):** A área de Forense obteve a segunda pior média da equipe (12,5/100). **87,5% da equipe desconhece a Ordem de Volatilidade (RFC 3227)**, prioritizando logs em disco antes da memória RAM em um incidente ativo, o que resultaria em destruição irreparável de evidências voláteis em cenários reais.
4. **Alinhamento Imediato com pwn.college:** O ambiente e a metodologia do **pwn.college** representam a solução ideal para os gaps encontrados. O modelo *hands-on* de engenharia reversa guiada, manipulação direta de registradores x86_64, exploração de shellcode e controle de fluxo é a intervenção cirúrgica necessária para transformar a Cyberguard em um time técnico de alta letalidade defensiva e ofensiva.

---

## 2. Diagnóstico Geral da Equipe

### 2.1 Desempenho Consolidado por Área Técnica

A pontuação média por área (escala 0 a 100) pondera o desempenho prático nas questões e a calibração de autoavaliação dos membros:

| Ranking | Área de Conhecimento | Média (0-100) | Nível Médio | Status de Risco | Taxa de Erro Médio |
| :---: | :--- | :---: | :---: | :---: | :---: |
| **1º** | **Engenharia Reversa e Binários** | **5.0** | Crítico | 🔴 Emergência | **93,75%** |
| **2º** | **Forense Digital e Resposta a Incidentes** | **12.5** | Crítico | 🔴 Emergência | **81,25%** |
| **3º** | **Infraestrutura e Active Directory** | **22.5** | Insuficiente | 🔴 Alto | **68,75%** |
| **4º** | **Prática e Resolução de CTF** | **25.0** | Insuficiente | 🟠 Atenção | **75,00%** |
| **5º** | **Blue Team (SIEM, Logs, Detecção)** | **26.2** | Insuficiente | 🟠 Atenção | **62,50%** |
| **6º** | **OSINT (Fontes Abertas)** | **26.2** | Insuficiente | 🟠 Atenção | **62,50%** |
| **7º** | **Criptografia Aplicada** | **27.5** | Insuficiente | 🟠 Atenção | **62,50%** |
| **8º** | **Web Pentest (OWASP Top 10)** | **32.5** | Básico | 🟠 Atenção | **62,50%** |
| **9º** | **Hardening e Defesa de Sistemas** | **35.0** | Básico | 🟡 Moderado | **50,00%** |
| **10º** | **Fundamentos (Linux, Windows, Redes)** | **37.1** | Básico | 🟡 Moderado | **54,17%** |
| **11º** | **Análise de Malware em Sandbox** | **47.5** | Intermediário | 🟡 Moderado | **37,50%** |
| **12º** | **Programação e Scripting (Python, C, Bash)** | **77.5** | Sólido | 🟢 Saudável | **0,00%** |

---

### 2.2 Raio-X Questão por Questão (Taxa de Acerto e Erro)

O inventário das 20 questões revela os gargalos exatos enfrentados pelo time:

| ID | Área | Pergunta Sintetizada | Acertos | Erros | % Acerto | % Erro | Causa Dominante de Erro |
| :---: | :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Q09** | Eng. Reversa | Função do registrador EIP/RIP em x86/x64 | **0** | **8** | **0,0%** | **100,0%** | 7 marcaram "Não sei"; 1 confundiu com EFLAGS |
| **Q18** | Forense Digital | Ordem de volatilidade (RFC 3227) / Memória RAM | **1** | **7** | **12,5%** | **87,5%** | 6 marcaram "Não sei"; 1 escolheu logs arquivados |
| **Q08** | Eng. Reversa | Diferença Análise Estática vs Dinâmica | **1** | **7** | **12,5%** | **87,5%** | 7 marcaram "Não sei"; apenas 1 acertou |
| **Q07** | Active Directory | Descrição e mecanismo do Kerberoasting | **2** | **6** | **25,0%** | **75,0%** | 5 marcaram "Não sei"; 1 confundiu com DoS em DNS |
| **Q17** | Forense Digital | Conceito de Cadeia de Custódia documental | **2** | **6** | **25,0%** | **75,0%** | 6 marcaram "Não sei" |
| **Q20** | CTF Prático | Reconhecimento de Base64 com padding `=` | **2** | **6** | **25,0%** | **75,0%** | 5 marcaram "Não sei"; 1 confundiu com Hash MD5 |
| **Q01** | Fundamentos | Permissão octal `chmod 640` no Linux | **3** | **5** | **37,5%** | **62,5%** | 4 marcaram "Não sei"; 1 marcou "leitura/escrita total" |
| **Q02** | Fundamentos | Three-Way Handshake TCP (SYN, SYN-ACK, ACK) | **3** | **5** | **37,5%** | **62,5%** | 5 marcaram "Não sei" |
| **Q05** | Web Pentest | Mitigação de SQLi (Prepared Statements) | **3** | **5** | **37,5%** | **62,5%** | 4 marcaram "Não sei"; 1 sugeriu blacklist de "SELECT" |
| **Q06** | Active Directory | Função do Domain Controller (DC) | **3** | **5** | **37,5%** | **62,5%** | 5 marcaram "Não sei" |
| **Q12** | Blue Team | Função principal de um SIEM | **3** | **5** | **37,5%** | **62,5%** | 5 marcaram "Não sei" |
| **Q13** | Blue Team | Definição da matriz MITRE ATT&CK | **3** | **5** | **37,5%** | **62,5%** | 5 marcaram "Não sei" |
| **Q16** | Criptografia | Hash lento com salt (bcrypt/Argon2) vs Reversível | **3** | **5** | **37,5%** | **62,5%** | 5 marcaram "Não sei" |
| **Q19** | OSINT | Exemplo de OSINT Passivo (WHOIS/DNS/Cert) | **3** | **5** | **37,5%** | **62,5%** | 5 marcaram "Não sei" |
| **Q11** | Análise Malware | Finalidade de packers (ofuscação/compressão) | **4** | **4** | **50,0%** | **50,0%** | 4 marcaram "Não sei" |
| **Q14** | Hardening | Definição de Hardening de Sistemas | **4** | **4** | **50,0%** | **50,0%** | 4 marcaram "Não sei" |
| **Q15** | Hardening | Hardening de SSH (desabilitar root/senha) | **4** | **4** | **50,0%** | **50,0%** | 4 marcaram "Não sei" |
| **Q03** | Fundamentos | Vulnerabilidade SSRF (Server-Side Request Forgery)| **5** | **3** | **62,5%** | **37,5%** | 3 marcaram "Não sei" |
| **Q10** | Análise Malware | Execução segura em VM isolada com snapshot | **6** | **2** | **75,0%** | **25,0%** | 2 marcaram "Não sei" |
| **Q04** | Programação | Insegurança da função `strcpy` em C | **8** | **0** | **100,0%** | **0,0%** | Nenhum erro registrado |

---

### 2.3 Matriz Geral de Desempenho por Membro

| Membro | Acertos / 20 | % Acerto | Eng. Reversa | DFIR | AD / Infra | CTF | Blue Team | Web | Fundam. | Scripting | Nível Classificado |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **Giovanni Mastelaro Marin** | **17/20** | **85%** | 1/2 | 1/2 | 2/2 | 1/1 | 2/2 | 0/1 | 3/3 | 1/1 | **Avançado / Liderança** |
| **Arthur G. T. Stellato** | **14/20** | **70%** | 0/2 | 2/2 | 2/2 | 0/1 | 0/2 | 1/1 | 3/3 | 1/1 | **Avançado / Infra** |
| **Silvio H. Mendes Dos Santos** | **14/20** | **70%** | 0/2 | 0/2 | 1/2 | 1/1 | 2/2 | 1/1 | 2/3 | 1/1 | **Avançado / Blue-CTF** |
| **Lauren Marçulo** | **9/20** | **45%** | 0/2 | 0/2 | 0/2 | 0/1 | 2/2 | 0/1 | 2/3 | 1/1 | **Intermediário / Blue** |
| **Joao Vitor Cordeiro** | **4/20** | **20%** | 0/2 | 0/2 | 0/2 | 0/1 | 0/2 | 0/1 | 1/3 | 1/1 | **Iniciante / Transição** |
| **Ismael R. Gonçalves Neto** | **2/20** | **10%** | 0/2 | 0/2 | 0/2 | 0/1 | 0/2 | 0/1 | 0/3 | 1/1 | **Iniciante / Base** |
| **Matheus Hirose Rodrigues** | **2/20** | **10%** | 0/2 | 0/2 | 0/2 | 0/1 | 0/2 | 1/1 | 0/3 | 1/1 | **Iniciante / Base** |
| **Luiz Paulo Tonini Guerra** | **1/20** | **5%** | 0/2 | 0/2 | 0/2 | 0/1 | 0/2 | 0/1 | 0/3 | 1/1 | **Iniciante / Base** |

---

## 3. Análise Segmentada por Perfis de Maturidade

A equipe divide-se nitidamente em **três cohorts de maturidade**, demandando abordagens andragógicas e planos de desenvolvimento técnico completamente distintos.

```
+-----------------------------------------------------------------------------------+
| COHORT 3: AVANÇADOS / VETERANOS (37,5%)                                           |
| Membros: Giovanni Marin (85%), Arthur Stellato (70%), Silvio Santos (70%)        |
| Características: Boa autonomia, dominam fundamentos e defesa, gaps em binários    |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
| COHORT 2: INTERMEDIÁRIO EM TRANSIÇÃO (12,5%)                                      |
| Membro: Lauren Marçulo (45%)                                                      |
| Características: Forte em Blue Team/Hardening/OSINT; zero em ofensiva/binários    |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
| COHORT 1: FUNDACIONAL / ONBOARDING CRÍTICO (50,0%)                                |
| Membros: Joao Vitor (20%), Matheus Hirose (10%), Ismael Neto (10%), Luiz Paulo(5%)|
| Características: Alta abstenção ("Não sei"), sem alicerce em redes/Linux/Web     |
+-----------------------------------------------------------------------------------+
```

---

### 3.1 Cohort 3: Avançados / Veteranos Técnicos (Aproveitamento 70% a 85%)

#### Perfil Coletivo
Integrantes com excelente discernimento de conceitos gerais de segurança da informação, familiaridade com ameaças modernas e raciocínio lógico estruturado. São os pilares técnicos para guiar os demais membros em programas de mentoria.
* **Gaps Notáveis:**
  * **Binários/Assembly:** Mesmo os líderes técnicos falharam em EIP/RIP. Giovanni foi o único de toda a Cyberguard a acertar análise estática vs dinâmica.
  * **Cegueira Pontual em Web e CTF:** Giovanni errou uma questão clássica de SQL Injection propondo uma mitigação via *blacklist* ingênua (`Bloquear a palavra "SELECT"`), um vício perigoso em segurança defensiva. Arthur errou o formato do Base64, confundindo padding `=` com hash MD5. Silvio errou a definição da permissão `640` no Linux.

---

### 3.2 Cohort 2: Intermediário em Transição (Aproveitamento 45%)

#### Perfil: Lauren Marçulo (9/20)
* **Perfil Coletivo:** Lauren demonstra sólida retenção e entendimento de conceitos de **Defesa, Monitoramento e Inteligência**: gabaritou SIEM, MITRE ATT&CK, Hardening geral, Hardening SSH e OSINT Passivo.
* **Gaps Notáveis:** Ausência completa de tração no universo de **Exploração Ofensiva, Active Directory, Criptografia, Forense e Binários**. Apresenta um perfil fortemente inclinado ao Blue Team e SOC, precisando de estímulo e base sólida para desenvolver visão adversária (*assume breach*).

---

### 3.3 Cohort 1: Fundacional / Onboarding Crítico (Aproveitamento 5% a 20%)

#### Perfil Coletivo: Joao Vitor Cordeiro, Matheus Hirose, Ismael Neto, Luiz Paulo Guerra
* **Perfil Coletivo:** Integrantes recém-ingressos ou que ainda não tiveram contato prático com laboratórios de cibersegurança. A esmagadora maioria de suas respostas foi a opção `E) Não sei/não conheço sobre o assunto`.
* **Aspecto Comportamental Positivo:** A altíssima taxa de resposta "Não sei" indica **honestidade intelectual e ausência de viés de chute aleatório**, o que gera uma linha de base fidedigna para treinamento.
* **Gaps Notáveis:** Falta de conhecimento dos blocos essenciais de computação: modelo TCP/IP, sistema de arquivos e permissões Linux, decodificações básicas e bancos de dados relacionais.

---

## 4. Dossiê Individual dos 8 Membros

---

### 4.1 Giovanni Mastelaro Marin
* **Score:** 17/20 (85,0%) | **Classificação:** Avançado / Nível Técnico Elevado
* **Pontos Fortes:** Dominou Fundamentos de Redes e Linux, Active Directory (DC e Kerberoasting), Blue Team (SIEM e MITRE), Hardening, Criptografia, OSINT e CTF. Foi o **único membro de toda a Cyberguard a acertar a diferenciação de análise estática vs dinâmica**.
* **Erros Cometidos (3/20):**
  1. *Web Pentest (SQLi):* Respondeu que a melhor mitigação é bloquear a palavra `"SELECT"`. **Gravidade Alta:** Sugerir filtragem por lista negra é uma das falhas conceituais mais clássicas do desenvolvimento web inseguro (facilmente contornável por *inline comments*, encoding ou payloads sem palavras reservadas diretas). A solução canônica é uso exclusivo de *prepared statements*.
  2. *Engenharia Reversa (EIP/RIP):* Afirmou que EIP/RIP armazena as *flags* das comparações (confundiu com EFLAGS/RFLAGS).
  3. *Forense Digital (Ordem de Volatilidade):* Afirmou que logs arquivados remotamente devem ser coletados antes da memória RAM. Desconsiderou a volatilidade crítica da DRAM ao desligar a máquina.
* **PDI (Plano de Desenvolvimento Individual):**
  * Alocar como mentor técnico do dojo pwn.college.
  * Módulos sugeridos: `Talking Web` e `Web Security` para sanar o conceito de injeções e parametrização; `Assembly Crash Course` para consolidar o mapa de registradores x86_64.

---

### 4.2 Arthur Gabriel Teotonio Stellato
* **Score:** 14/20 (70,0%) | **Classificação:** Avançado / Perfil Infra & Defesa
* **Pontos Fortes:** Gabaritou Fundamentos (Linux 640, TCP, SSRF), Active Directory completo (DC e Kerberoasting), Hardening (Geral e SSH), Criptografia (bcrypt/salt) e Forense Digital (foi o **único a acertar a Ordem de Volatilidade e Memória RAM**).
* **Erros Cometidos (6/20):**
  1. *Engenharia Reversa:* Marcou "Não sei" para análise estática x dinâmica e para o registrador EIP/RIP.
  2. *Blue Team:* Marcou "Não sei" tanto para SIEM quanto para a matriz MITRE ATT&CK.
  3. *OSINT:* Marcou "Não sei" para a identificação de OSINT passivo.
  4. *CTF:* Afirmou que a string em Base64 `ZmxhZ3t0ZXN0ZX0=` era um Hash MD5 (MD5 possui 32 caracteres hexadecimais, sem padding `=` e sem letras maiúsculas além de A-F).
* **PDI:**
  * Imersão direta em Engenharia Reversa prática no pwn.college.
  * Treinamento rápido de Threat Intelligence (MITRE ATT&CK Enterprise Matrix) e reconhecimento de artefatos de encoding vs hashing.

---

### 4.3 Silvio Henrique Mendes Dos Santos
* **Score:** 14/20 (70,0%) | **Classificação:** Avançado / Perfil Operacional & SOC
* **Pontos Fortes:** Excelente desempenho em Blue Team (SIEM, MITRE), Hardening, Criptografia, OSINT, Sandbox de Malware (prática e packers), Web Pentest (SQLi parametrizado) e CTF (reconheceu Base64 imediatamente).
* **Erros Cometidos (6/20):**
  1. *Fundamentos (Permissões Linux):* Respondeu que `chmod 640` confere leitura e escrita para todos os usuários (confundiu com 666 ou 777). Em ambiente produtivo, isso representaria uma brecha grave de privilégios.
  2. *Active Directory (Kerberoasting):* Afirmou que o Kerberoasting consiste em "esgotar o servidor DNS do domínio" (confundiu com Denial of Service / DNS amplification).
  3. *Engenharia Reversa:* Marcou "Não sei" nas duas questões (análise estática/dinâmica e EIP/RIP).
  4. *Forense Digital:* Marcou "Não sei" para cadeia de custódia e ordem de volatilidade.
* **PDI:**
  * Revisão prática de modelo de permissões Unix (rwx em binário e octal, umask, sticky bit).
  * Módulos práticos de Active Directory Attacks (BloodHound, Impacket, Kerberoasting) e Forense de Memória (Volatility Workbench).
  * Trilha de introdução ao Assembly no pwn.college.

---

### 4.4 Lauren Marçulo
* **Score:** 9/20 (45,0%) | **Classificação:** Intermediário / Vocação Blue Team & GRC
* **Pontos Fortes:** Demonstrou 100% de acerto em Blue Team (SIEM e MITRE ATT&CK), Hardening (Sistemas e SSH) e OSINT passivo. Acertou a permissão octal `chmod 640` no Linux e a mitigação conceitual de `strcpy`.
* **Erros Cometidos (11/20):**
  1. *Redes e Web:* Não soube o handshake TCP (SYN-ACK) nem a mitigação de SQLi.
  2. *Active Directory:* Não soube o papel do DC nem o conceito de Kerberoasting.
  3. *Engenharia Reversa e Forense:* Lacuna completa (marcou "Não sei" em todas as 4 questões).
  4. *Criptografia e CTF:* Não soube hash lento/salt nem a identificação de Base64.
* **PDI:**
  * Potencial expressivo para a ala defensiva da Cyberguard.
  * Necessita de nivelamento em Redes de Computadores (Wireshark, TCP/IP) e Segurança de Aplicações Web.
  * Inserção em desafios graduais no pwn.college (`Program Misuse` e `Talking Web`).

---

### 4.5 Joao Vitor Cordeiro
* **Score:** 4/20 (20,0%) | **Classificação:** Iniciante em Transição
* **Pontos Fortes:** Demonstrou intuição acima da média do grupo iniciante ao acertar a identificação de vulnerabilidade **SSRF** e ao acertar **ambas as questões de Análise de Malware** (VM isolada com snapshot e uso de packers para ofuscação/compressão). Acertou `strcpy`.
* **Erros Cometidos (16/20):**
  * Marcou "Não sei" em todas as questões de Redes (TCP handshake), Permissões Linux, SQL Injection, Active Directory, Engenharia Reversa, Blue Team, Hardening, Criptografia, Forense, OSINT e CTF.
* **PDI:**
  * Demonstra afinidade com comportamento de código malicioso e sandbox.
  * Prioridade absoluta: Fundamentos de Sistemas Operacionais, Redes e Linux CLI.

---

### 4.6 Matheus Hirose Rodrigues
* **Score:** 2/20 (10,0%) | **Classificação:** Iniciante / Base
* **Pontos Fortes:** Acertou a mitigação de **SQL Injection via Prepared Statements** (superando inclusive membros veteranos que sugeriram filtros manuais) e a questão conceitual sobre `strcpy`.
* **Erros Cometidos (18/20):**
  * Marcou "Não sei" para as outras 18 questões do exame, revelando ausência de bagagem prática em laboratórios e ferramentas.
* **PDI:**
  * Iniciar pelo Dojo de Fundamentos do pwn.college: manipulação de shell Linux, pipes, redirecionamentos e arquitetura básica de redes.

---

### 4.7 Ismael Rodrigues Gonçalves Neto
* **Score:** 2/20 (10,0%) | **Classificação:** Iniciante / Base
* **Pontos Fortes:** Acertou que a análise dinâmica de malware deve ser feita em VM isolada com snapshots e sem conexão com produção; acertou o perigo de `strcpy`.
* **Erros Cometidos (18/20):**
  * Marcou "Não sei" em 18 questões consecutivas.
* **PDI:**
  * Participar obrigatoriamente do ciclo de onboarding: Git, Linux Essentials, Redes Básicas e CTF para iniciantes (picoCTF / pwn.college `Baby's First`).

---

### 4.8 Luiz Paulo Tonini Guerra
* **Score:** 1/20 (5,0%) | **Classificação:** Iniciante / Base
* **Pontos Fortes:** Acertou unicamente a questão sobre a função `strcpy` em C.
* **Erros Cometidos (19/20):**
  * Marcou "Não sei" em todas as demais 19 questões do questionário diagnóstico.
* **PDI:**
  * Requer acompanhamento de mentoria individualizada (*buddy system*).
  * Foco imediato na Trilha Zero: Fundamentos de Arquitetura de Computadores, Shell Scripting, Comandos GNU/Linux e Protocolos de Rede.

---

## 5. Análise de Causa-Raiz das Maiores Dificuldades

```
+-----------------------------------------------------------------------------------------------------+
|                                ÁRVORE DE CAUSA-RAIZ DOS MAIORES GAPS                                |
+-----------------------------------------------------------------------------------------------------+
|                                                                                                     |
|  [0% Acerto em EIP/RIP]        [87,5% Erro em Forense/RAM]       [62,5% Erro em Permissão/TCP]      |
|           |                                 |                                 |                     |
|           v                                 v                                 v                     |
|  Abstração excessiva           Cultura reativa sem vivência     Ensino acadêmico desconectado       |
|  de linguagens de alto nível   de incidentes reais; desconhe-   da prática em terminal e da         |
|  (Python/JS); ausência de      cimento do padrão RFC 3227       análise de pacotes via Wireshark/   |
|  depuração prática em GDB.     (coleta volátil de evidências).  tcpdump.                            |
|                                                                                                     |
+-----------------------------------------------------------------------------------------------------+
```

---

### 5.1 O Paradoxo de Binários: O "Efeito strcpy" vs Ausência de Baixo Nível (EIP/RIP)
O contraste mais dramático do relatório reside no fato de que **100% dos membros souberam que `strcpy` causa estouro de buffer**, mas **0% souberam o que é o registrador EIP/RIP**.

#### Causa-Raiz Técnica
* **Conhecimento Decorado vs Mecânica de Execução:** Em cursos de ciência da computação ou tutoriais superficiais de desenvolvimento seguro, repete-se como um mantra que `strcpy()` é inseguro e deve ser substituído por `strncpy()` ou `snprintf()`. No entanto, os alunos raramente são levados a inspecionar a memória no GDB para responder: *O que acontece fisicamente quando o buffer transborda?*
* **Desconhecimento da Pilha (Stack Frame):** Os membros não visualizam o *buffer*, o *Saved Frame Pointer* (RBP) e o endereço de retorno (*Saved Return Address*). Eles desconhecem que o registrador **EIP** (Extended Instruction Pointer, 32 bits) ou **RIP** (Register Instruction Pointer, 64 bits) armazena o ponteiro da próxima instrução a ser decodificada pelo processador. Sem entender o RIP, o membro é incapaz de compreender *buffer overflows*, *ROP chains*, *shellcode execution* ou defesas modernas como ASLR, DEP/NX e Stack Canaries.

---

### 5.2 A Falha Estrutural em Fundamentos: `chmod 640` e Handshake TCP
Mais de 62% da equipe falhou em identificar o significado de `chmod 640` e a ordem do *three-way handshake* TCP (`SYN`, `SYN-ACK`, `ACK`).

#### Causa-Raiz Técnica
* **Interface Gráfica vs Terminal Headless:** Muitos ingressantes na área de segurança utilizam distribuições prontas (como Kali Linux) via interface gráfica ou copiam comandos da internet sem decompor os argumentos numéricos. 
* O cálculo da máscara octal (`6` = Leitura 4 + Escrita 2; `4` = Leitura 4; `0` = Sem acesso) exige raciocínio de bits de permissão (`rw-r-----`) nas tríades Usuário, Grupo e Outros. O erro de Silvio Santos (afirmando que dava acesso total a todos) demonstra confusão entre `640` e `666/777`.
* **Redes sem Captura Real:** O *three-way handshake* é decorado para provas teóricas, mas esquecido se o estudante nunca abriu o **Wireshark** ou o **tcpdump** para filtrar uma conexão `tcp.flags.syn == 1`. O entendimento desse handshake é mandatório para ataques de rede (SYN Flood, Port Scanning, RST injection, Man-in-the-Middle).

---

### 5.3 A Ilusão Forense e a Violação da RFC 3227 (Ordem de Volatilidade)
7 dos 8 membros erraram a ordem de coleta de evidências em uma máquina comprometida. Giovanni sugeriu coletar logs arquivados remotamente antes da memória local.

#### Causa-Raiz Técnica
* **Intuicionismo Falho:** A intuição leiga sugere que arquivos e logs em disco são a principal fonte de investigação. No entanto, de acordo com a **RFC 3227** (*Guidelines for Evidence Collection and Archiving*), a coleta deve seguir estritamente o critério de tempo de vida da informação:
  1. Registradores da CPU e memória cache.
  2. **Memória de acesso aleatório (RAM)** (chaves de criptografia em memória, conexões de rede ativas, processos injetados em memória que não tocaram o disco - *fileless malware*).
  3. Estados de rede e tabelas de roteamento/ARP.
  4. Sistemas de arquivos e discos permanentes.
  5. Mídias arquivadas remotamente e logs estáticos.
* Ao desligar o computador ou esperar para coletar a RAM, perde-se a chave do Ransomware, o artefato na memória do processo e a conexão reversa do C2 (*Command & Control*).

---

### 5.4 Active Directory e Ataques Corporativos Modernos (Kerberoasting)
6 dos 8 membros não souberam definir Kerberoasting, e um dos veteranos confundiu com negação de serviço via DNS.

#### Causa-Raiz Técnica
* **Foco Excessivo em Linux / Web em detrimento do Ecossistema Windows Corporativo:** O ambiente de ensino da maioria dos estudantes e CTFs iniciantes foca excessivamente em Linux e vulnerabilidades Web simples. No entanto, mais de 90% das redes corporativas globais operam sob **Active Directory / Entra ID**.
* O **Kerberoasting** é um dos ataques mais prevalentes do cenário contemporâneo: qualquer usuário autenticado no domínio (mesmo sem privilégios) pode solicitar ao KDC um Ticket Granting Service (TGS) para qualquer conta com *Service Principal Name* (SPN) registrado e quebrar o hash RC4/AES offline via hashcat. O desconhecimento desse vetor impede que a equipe atue eficientemente tanto em Red Team quanto na auditoria de contas de serviço.

---

### 5.5 Identificação de Artefatos em CTFs (Base64 vs MD5)
6 dos 8 membros erraram a questão da string `ZmxhZ3t0ZXN0ZX0=`. Arthur supôs tratar-se de um hash MD5.

#### Causa-Raiz Técnica
* **Ausência de Prática com Encoding e Decodificação:** Um hash MD5 gera uma saída hexadecimal fixa de 128 bits (32 caracteres contendo apenas `0-9` e `a-f`). A presença de letras maiúsculas misturadas (`Z`, `m`, `x`, `Y`) e o caractere de preenchimento (*padding*) `=` no final é a assinatura inequívoca de codificação **Base64** (RFC 4648). A string decodificada revela literalmente a flag: `flag{teste}`.
* Esse gap demonstra que a equipe carece de vivência diária em desafios estilo Jeopardy de CTF, onde o reconhecimento visual rápido de assinaturas de encoding é a primeira habilidade exigida.

---

## 6. Matriz de Competências & Alinhamento Estratégico com pwn.college

O **pwn.college** (desenvolvido pelo Arizona State University e mantido por hackers de renome mundial da equipe Shellphish) é a plataforma pedagógica mais avançada do mundo para sanar exatamente as deficiências expostas neste relatório. Diferente de plataformas puramente gamificadas, o pwn.college aborda a computação do silício à aplicação web.

### 6.1 Mapeamento Direto: Gaps do Diagnóstico ➔ Módulos pwn.college

| Gap Crítico Identificado | Módulo Recomendado no pwn.college | Habilidades Desenvolvidas no Módulo |
| :--- | :--- | :--- |
| **Registradores, EIP/RIP, Baixo Nível** | `Assembly Crash Course` | Registradores x86_64 (RAX, RBX, RIP, RSP, RBP), instruções fundamentais, fluxo de controle, saltos condicionais. |
| **Exploração de Binários & Buffer Overflow** | `Memory Errors` & `Return Oriented Programming` | Transbordamento de pilha, sobrescrita de Saved RIP, controle do fluxo de execução, bypass de proteções. |
| **Fundamentos Linux, Permissões e Arquivos** | `Program Misuse` & `Linux Luminarium` | Permissões POSIX, SUID bits, manipulação de descritores de arquivo, bypass de restrições de privilégios. |
| **Interação Web Insegura & SSRF** | `Talking Web` & `Web Security` | Construção artesanal de requisições HTTP, headers, SSRF, injection flaws, interação cliente-servidor. |
| **Decodificação de Dados & Scripting de Exploits**| `Building a Web Server` & `Shellcode Injection` | Manipulação de bytes crus, encodings (Base64, Hex, URL), injeção de instruções de máquina em memória. |
| **Engenharia Reversa Estática e Dinâmica** | `Reverse Engineering` | Uso de descompiladores (Ghidra), depuração dinâmica com GDB/GEF, análise de fluxo de controle binário. |

---

### 6.2 Desenho das Trilhas de Capacitação na Cyberguard

```
                  TRILHA 1: FUNDAMENTOS & LINUX LUMINARIUM
                  (Público: Cohort 1 - Luiz Paulo, Ismael, Matheus, Joao)
                                      |
                                      v
                  TRILHA 2: ARQUITETURA X86_64 & ASSEMBLY
                  (Público: Todos os membros - Obrigatório)
                                      |
                                      +------------------------------------+
                                      |                                    |
                                      v                                    v
             TRILHA 3: OFENSIVA & BINARY (PWN)             TRILHA 4: DEFENSIVA & FORENSE (DFIR)
             (Giovanni, Arthur, Silvio, Joao)              (Lauren, Silvio, Arthur, Matheus)
             - pwn.college: Memory Errors                  - Volatility Memory Analysis
             - Return Oriented Programming                 - SIEM & Elastic Detection Rules
             - Active Directory Hands-on                   - MITRE ATT&CK Mapping
```

#### Trilha A — Fundamentos Operacionais (Obrigatória para Cohort 1)
* **Objetivo:** Nivelar o time de base em ferramentas Unix, shell scripting e redes.
* **Módulos:**
  1. *Linux Luminarium (pwn.college):* Comandos básicos, redirecionamentos (`<`, `>`, `|`), variáveis de ambiente e permissões (`chmod`, `chown`, `umask`).
  2. *Networking Fundamentals:* Análise prática de tráfego com Wireshark; dissecação do three-way handshake e cabeçalhos IP/TCP/UDP.
  3. *Encoding & Crypto Basics:* Diferenciação prática de Base64, Hexadecimal, Hashes rápidos (MD5/SHA256) vs Hashes com salt/fator de custo (bcrypt/Argon2).

#### Trilha B — Arquitetura de Sistemas & Engenharia Reversa (Geral / Emergencial)
* **Objetivo:** Extinguir a nota 5.0/100 em Engenharia Reversa e o erro em EIP/RIP.
* **Módulos:**
  1. *Assembly Crash Course (pwn.college):* Prática com registradores gerais e especiais (`RIP`, `RSP`, `RBP`, `RFLAGS`).
  2. *Depuração Prática:* Uso do GDB com extensão GEF (*GDB Enhanced Features*). Breakpoints, visualização da pilha em tempo real e registradores.
  3. *Análise Estática vs Dinâmica:* Uso de `strings`, `ltrace`, `strace` e introdução ao **Ghidra**.

#### Trilha C — Infraestrutura Corporativa & Active Directory (Avançados + Intermediários)
* **Objetivo:** Capacitar a equipe em topologias Windows Domain / Active Directory.
* **Laboratórios Práticos:**
  1. Subida de lab local via Vagrant/Proxmox com Windows Server 2022 (Domain Controller) e 2 Windows Workstations.
  2. Execução manual de Kerberoasting (`GetUserSPNs.py` da suite Impacket e quebra via `hashcat -m 13100`).
  3. Extração e análise de tickets Kerberos (TGT / TGS).

#### Trilha D — Forense Digital & Resposta a Incidentes (DFIR)
* **Objetivo:** Reverter a deficiência em coleta volátil e cadeia de custódia.
* **Módulos:**
  1. *Live Forensics & RFC 3227:* Coleta de memória em hosts comprometidos utilizando `LiME` (Linux) e `WinPmem` (Windows).
  2. *Volatility 3:* Análise da imagem de memória (plugins: `windows.pslist`, `windows.netscan`, `windows.malfind`).
  3. *Cadeia de Custódia:* Procedimentos de hash criptográfico de integridade (`sha256sum`) e termo formal de custódia.

---

## 7. Roadmap Operacional e Metas de Curto, Médio e Longo Prazo

Para garantir a evolução mensurável da Cyberguard até o final do ciclo 2026.2, estabelece-se o seguinte cronograma de sprints:

### 7.1 Cronograma de Execução (Semanas 1 a 12)

```
Semanas 01-02: Setup do Dojo pwn.college + Nivelamento Linux & Redes (Wireshark + chmod)
Semanas 03-04: Assembly Crash Course no pwn.college (Dominando Registradores e RIP/EIP)
Semanas 05-06: Program Misuse & Memory Errors (Buffer Overflows Reais no pwn.college)
Semanas 07-08: Web Security & Talking Web (SSRF, SQLi Parametrizado, Encoding Base64)
Semanas 09-10: Laboratório Hands-on de Active Directory (Kerberoasting + Blue Team SIEM)
Semanas 11-12: Simulado Geral CTF Interno + Novo Teste Diagnóstico de Saída (Re-avaliação)
```

| Fase | Período | Ação Principal | Entregável / Meta Tangível |
| :--- | :--- | :--- | :--- |
| **Fase 1: Fundação** | Semanas 1 e 2 | Bootcamp de Linux, Redes e Ferramental de CTF | 100% dos membros completam o módulo inicial de Linux; 0 erros em `chmod` e handshake TCP. |
| **Fase 2: Imersão em Binários** | Semanas 3 e 4 | Imersão obrigatória no `Assembly Crash Course` (pwn.college) | Cada membro resolve no mínimo 25 desafios práticos de Assembly manipulando registradores. |
| **Fase 3: Exploração de Memória** | Semanas 5 e 6 | Laboratório `Memory Errors` do pwn.college | Execução de buffer overflow controlado redirecionando o fluxo de controle via RIP no GDB. |
| **Fase 4: Web & Active Directory** | Semanas 7 e 8 | Treinamento Web OWASP e Lab prático de AD | Simulação de Kerberoasting em lab isolado com captura de TGS e mitigação de SQLi com PDO/Prepared Statements. |
| **Fase 5: Forense & Defesa** | Semanas 9 e 10 | Workshop de DFIR e Blue Team | Dump prático de memória RAM, análise com Volatility e mapeamento de ataque na matriz MITRE ATT&CK. |
| **Fase 6: Consolidação** | Semanas 11 e 12| CTF Interno Cyberguard & Re-avaliação Diagnóstica | Aplicação do Teste Diagnóstico Final 2026.2 para comprovação de salto estatístico de performance. |

---

### 7.2 Metas Numéricas (KPIs de Sucesso)

1. **Eliminação do "Efeito 0%":** Elevar a taxa de acerto no registrador EIP/RIP de **0% para 100%** de aprovação no teste de reavaliação.
2. **Engenharia Reversa:** Elevar a média da equipe de **5.0/100 para no mínimo 65.0/100**.
3. **Forense e DFIR:** Elevar a média da equipe de **12.5/100 para no mínimo 70.0/100**.
4. **Média Geral da Equipe:** Saltar dos atuais **39,38% para uma média mínima de 75,0%** de acertos globais.
5. **Engajamento no pwn.college:** Todo integrante deverá atingir, no mínimo, a faixa amarela (*belt*) ou equivalente no ranking interno da Cyberguard.

---

## 8. Parecer Final e Recomendações à Coordenação

O resultado do Teste Diagnóstico 2026.2 não deve ser interpretado como um atestado de incapacidade, mas sim como um **mapa cirúrgico de prioridades**. A equipe possui três membros veteranos de alto potencial (Giovanni, Arthur e Silvio) que conseguem tracionar o grupo, uma integrante vocacionada para a vertente defensiva (Lauren) e quatro membros que representam uma página em branco altamente receptiva.

O modelo tradicional de ensino expositivo falhou em transmitir o funcionamento da máquina em baixo nível, gerando profissionais que conhecem nomes de vulnerabilidades (como buffer overflow e strcpy), mas não sabem como a CPU as executa. A adoção irrestrita da metodologia do **pwn.college**, complementada por laboratórios de Active Directory e Forense Digital, colocará a Cyberguard em um patamar de excelência técnica reconhecido nacionalmente no cenário universitário e de segurança ofensiva/defensiva.

---
*Relatório concluído e homologado pela Divisão de Inteligência Técnica — Cyberguard Hub.*
