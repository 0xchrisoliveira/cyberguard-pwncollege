# Guia Oficial do Dojo e Treinamento Cyberguard no pwn.college
## Configuração do Dojo, Acompanhamento da Equipe & Roteiro de Estudos (Ciclo 2026.2)

---

## 1. Dojos no pwn.college: A Nova Estrutura de Equipes

Nas versões contemporâneas do **pwn.college**, o conceito tradicional de *"Crews"* foi unificado e potencializado dentro da arquitetura de **Dojos Customizados**. 

Em vez de apenas um time em um placar global genérico, um **Dojo próprio** oferece um ambiente de treinamento completo e exclusivo:
* **Placar Interno Dedicado (Scoreboard Exclusivo):** Todos os membros que acessam o Dojo da Cyberguard competem e colaboram em um ranking próprio do time, com métricas de desafios resolvidos por módulo.
* **Painel Administrativo da Liderança (`Admin`):** Os líderes técnicos da Cyberguard têm visão analítica do progresso dos integrantes desafio a desafio, podendo identificar exatamente onde cada membro travou.
* **Currículo Sob Medida:** O conteúdo é carregado diretamente do repositório GitHub do time, permitindo que as explicações teóricas e desafios estejam em Português (PT-BR) e alinhados às necessidades da equipe.

---

## 2. Passo a Passo: Ativação e Entrada no Dojo da Cyberguard

```
[ PASSO 1 ]                    [ PASSO 2 ]                  [ PASSO 3 ]                  [ PASSO 4 ]
Liderança cria o Dojo       -> Membros criam conta      -> Membros acessam a URL     -> Progresso automático
via GitHub Repo                no pwn.college               do Dojo da Cyberguard        no Scoreboard do time
```

### Para a Liderança (Criação e Gestão do Dojo):
1. Faça login em [pwn.college](https://pwn.college).
2. Acesse: **[https://pwn.college/dojos/create](https://pwn.college/dojos/create)** (ou clique em **Dojos** > **Create Dojo**).
3. No campo **Repository**, informe a URL pública do nosso repositório:
   ```text
   https://github.com/0xchrisoliveira/cyberguard-pwncollege
   ```
4. Clique em **Create Dojo**. O pwn.college importará o `dojo.yml` e gerará o Dojo oficial da Cyberguard.
5. Copie a URL gerada (ex: `https://pwn.college/cyberguard-ethical-hacking`) e compartilhe com o time.

### Para os Membros do Time:
1. Acesse [pwn.college](https://pwn.college) e crie sua conta (ou faça login com GitHub).
2. Recomendamos usar seu nome ou handle habitual da Cyberguard como nome de usuário.
3. No canto superior direito, acesse **Settings** e cadastre sua chave pública SSH no campo **"SSH Public Key"** (para poder conectar via terminal se desejar).
4. Acesse a URL do Dojo da Cyberguard fornecida pela liderança e inicie pelo **Módulo 1**.
5. Não é necessária nenhuma ação manual para sincronização: toda flag submetida atualiza imediatamente o placar do time!

---

## 4. Roteiro de Estudos Personalizado por Cohort

Com base no diagnóstico do ciclo 2026.2, identificamos uma equipe com assimetria de competências, dividida em três perfis bem delimitados. Cada grupo possui prioridades, ritmos e metas específicas para que todos atinjam a excelência técnica.

---

### 🛡️ Trilha 1: Onboarding Fundacional (Cohort 1)
**Membros:** Luiz Paulo Tonini Guerra, Ismael Rodrigues Gonçalves Neto, Matheus Hirose Rodrigues, Joao Vitor Cordeiro.  
**Média de Entrada:** 5% a 20% de aproveitamento.  
**Diagnóstico:** Integrantes com excelente honestidade intelectual (alta taxa de abstenção "Não sei" em vez de chutes), demonstrando intuição positiva (ex: Matheus acertou mitigação de SQLi; Joao Vitor acertou conceitos de malware e SSRF), mas sem base consolidada em comandos Linux, redes ou permissões.

#### Objetivos Primários:
1. Perder o medo do terminal Linux e dominar caminhos, pipes, redirecionamentos e cálculo de permissões octais (`chmod 640`, `755`).
2. Entender como computadores conversam na rede (Three-Way Handshake: `SYN` -> `SYN-ACK` -> `ACK`).
3. Desenvolver pequenos scripts de automação em Python (leitura de strings e conexão por sockets).

#### Cronograma Semanal Recomendado:
* **Semanas 1 e 2:** **Módulo 1 (`fundamentos-linux`)**
  - Desafios: `linux-nav-files`, `linux-io-redirection`, `linux-env-manipulation`, `linux-permissions-octal`, `linux-suid-privilege`.
  - *Meta:* Conseguir calcular na ponta da língua por que `chmod 640` é leitura/escrita para o dono, leitura para o grupo e nada para outros.
* **Semanas 3 e 4:** **Módulo 2 (`redes-protocolos`)**
  - Desafios: `net-nc-raw-socket`, `net-tcpdump-sniffing`, `net-tcp-handshake-analysis`, `net-nmap-recon`.
  - *Meta:* Capturar e analisar o fluxo SYN/SYN-ACK/ACK no Wireshark/tcpdump.
* **Semanas 5 e 6:** **Módulo 3 (`scripting-automacao`)** & **Módulo 5 (`criptografia-ctf-basics`)**
  - Desafios: `script-bash-loop`, `script-python-socket-math`, `crypto-encodings-pipeline`.
  - *Meta:* Saber diferenciar imediatamente uma string Base64 (`=`) de um hash MD5 ou SHA-256 e escrever seu primeiro script de socket em Python.

---

### 🛡️ Trilha 2: Intermediário em Transição / Blue & DevSec (Cohort 2)
**Membro:** Lauren Marçulo.  
**Aproveitamento de Entrada:** 45%.  
**Diagnóstico:** Desempenho impecável em Blue Team, SIEM, MITRE ATT&CK, Hardening de Servidores/SSH e OSINT Passivo. Acertou permissões Unix (`chmod 640`). Gaps concentrados na visão ofensiva adversária, redes profundas, injeções web e engenharia reversa.

#### Objetivos Primários:
1. Conectar a teoria de defesa e monitoramento com a perspectiva do atacante (*assume breach*).
2. Compreender a fundo o funcionamento dos protocolos de rede e falhas de aplicação web (SQLi via prepared statements, SSRF).
3. Iniciar o contato com a arquitetura de processadores x86_64 e análise de memória volátil (DFIR).

#### Cronograma Semanal Recomendado:
* **Semana 1:** **Módulo 2 (`redes-protocolos`)**
  - Consolidar a análise de pacotes e o ciclo de vida de conexões TCP/IP.
* **Semanas 2 e 3:** **Módulo 4 (`web-pentest-owasp`)**
  - Desafios: `web-http-headers-cookies`, `web-sqli-login-bypass`, `web-sqli-remediation`, `web-ssrf-internal-recon`.
  - *Meta:* Dominar a remediação prática de SQL Injection com *prepared statements* e entender a gravidade do SSRF em ambientes de nuvem e intranet.
* **Semana 4:** **Módulo 6 (`forense-defesa`)**
  - Desafios: `dfir-volatility-order`, `dfir-memory-dump-inspection`, `dfir-log-correlation-mitre`.
  - *Meta:* Aplicar a Ordem de Volatilidade da RFC 3227 extraindo artefatos da memória RAM e correlacionando eventos com a matriz MITRE ATT&CK.
* **Semanas 5 e 6:** **Módulo 8 (`engenharia-reversa-x86`)**
  - *Meta:* Desmistificar a CPU, entender o registrador RIP e a diferença entre análise estática e dinâmica no GDB.

---

### 🛡️ Trilha 3: Veteranos Técnicos & Liderança Ofensiva (Cohort 3)
**Membros:** Giovanni Mastelaro Marin, Arthur Gabriel Teotonio Stellato, Silvio Henrique Mendes Dos Santos.  
**Média de Entrada:** 70% a 85% de aproveitamento.  
**Diagnóstico:** Domínio robusto de conceitos gerais, infraestrutura corporativa, Active Directory e segurança defensiva. Giovanni foi o único a acertar a distinção entre análise estática e dinâmica; Arthur foi o único a dominar a RFC 3227.  
**Ponto Cego Crítico da Liderança:** 100% dos veteranos erraram o papel do registrador EIP/RIP, e falhas pontuais ocorreram em SQLi (tentativa de mitigação por blacklist), permissões e identificação de formatos Base64.

#### Objetivos Primários:
1. Superar o "abismo do baixo nível": dominar a arquitetura x86_64, a função de RIP/RSP/RBP e a pilha de execução (*stack frame*).
2. Conquistar fluência em exploração de binários (*pwn*): estouro de buffer (*stack buffer overflow*), cálculo de offset e sequestro de fluxo de execução.
3. Atuar formalmente como mentores técnicos dos membros das Trilhas 1 e 2.

#### Cronograma Semanal Recomendado:
* **Semana 1:** **Nivelamento Relâmpago e Remoção de Vícios**
  - Revisar e concluir os desafios avançados dos Módulos 1 a 6 (ênfase em `linux-suid-privilege`, `web-sqli-remediation`, `dfir-memory-dump-inspection` e `ad-kerberoasting-offline`).
  - Corrigir concepções sobre filtragem por blacklist e fixar o formato canônico de Base64 e hashes com salt.
* **Semanas 2 e 3:** **Módulo 8 (`engenharia-reversa-x86`)**
  - Desafios: `rev-static-strings-objdump`, `rev-gdb-rip-inspection`, `rev-control-flow-cmp-jmp`, `rev-ghidra-decompilation`.
  - *Meta:* Inspecionar o registrador RIP passo a passo no GDB e reconstruir fluxos condicionais descompilados no Ghidra.
* **Semanas 4, 5 e 6:** **Módulo 9 (`exploracao-binaria-pwn`)**
  - Desafios: `pwn-stack-anatomy-crash`, `pwn-win-function-redirection`, `pwn-shellcraft-injection`, `pwn-capstone-challenge`.
  - *Meta:* Compreender a vulnerabilidade estrutural de `strcpy`/`gets`, calcular o offset exato até o RIP salvo e redirecionar a execução para um shellcode interativo.

---

## 5. Dinâmica Operacional e Rituais de Treinamento

Para assegurar a evolução contínua da Cyberguard, adotaremos os seguintes rituais semanais:

1. **Sprint Semanal no pwn.college:**
   - Toda segunda-feira é definido o foco temático da semana no Discord do time.
   - Cada membro estabelece uma meta mínima de 2 a 3 desafios por semana de acordo com sua trilha.
2. **Mentoria em Pares (Pair Hacking):**
   - Os membros da Trilha 3 (Giovanni, Arthur, Silvio) farão duplas rotativas semanais com os membros da Trilha 1 e 2 para sessões ao vivo de 45 minutos no Web Terminal ou VS Code integrado.
   - Regra de ouro da mentoria: **O mentor guia pelo raciocínio, quem digita os comandos é o mentorado.**
3. **Debriefing Semanal e Write-ups Internos:**
   - Aos finais de semana, realizamos uma sessão técnica rápida (Sync de 1 hora) onde um membro de cada Trilha apresenta a resolução de um desafio marcante que concluiu.
   - Os membros são estimulados a documentar suas resoluções em um repositório interno de notas (`Write-ups Cyberguard`).

---

## 6. Checklist de Início Rápido

- [ ] Conta criada e confirmada no `pwn.college`
- [ ] Chave pública SSH adicionada em `Settings -> SSH Public Key`
- [ ] Ingresso confirmado na Crew **"Cyberguard"** na aba `/crews`
- [ ] Primeiro desafio do Módulo 1 (`linux-nav-files`) concluído no terminal
- [ ] Primeira flag submetida e pontuada no placar da equipe!

*Bem-vindo à Crew Cyberguard no pwn.college. Vamos em frente!*
