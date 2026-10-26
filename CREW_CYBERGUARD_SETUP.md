# Guia Oficial da Crew Cyberguard no pwn.college
## Configuração, Dinâmica de Time & Roteiro de Estudos Personalizado (Ciclo 2026.2)

---

## 1. O que é uma Crew no pwn.college?

No ecossistema do **pwn.college**, uma **Crew** (Equipe) é uma agremiação colaborativa oficial de membros que compartilham uma identidade comum, acompanham o progresso mútuo e somam esforços em um placar (*scoreboard*) unificado.

Diferente de competições individuais onde cada um luta isoladamente, a Crew da **Cyberguard** foi concebida sob o lema:
> *"Nenhum operador fica para trás. A força da nossa defesa e o impacto do nosso ataque são medidos pela solidez do nosso elo mais jovem."*

### Benefícios de Participar da Crew:
* **Placar Coletivo e Visibilidade:** Toda flag submetida por qualquer membro da Cyberguard pontua para a reputação geral do nosso time no ranking da plataforma.
* **Acompanhamento de Evolução:** Os líderes técnicos e mentores conseguem visualizar quais desafios os membros estão enfrentando dificuldades em tempo real, permitindo intervenções didáticas cirúrgicas.
* **Cultura de Mentoria:** Pareamento estruturado entre veteranos e iniciantes para acelerar a curva de aprendizado.

---

## 2. Passo a Passo: Entrando na Crew Cyberguard

Siga o procedimento abaixo para vincular sua conta à nossa equipe oficial:

```
[ PASSO 1 ]             [ PASSO 2 ]             [ PASSO 3 ]             [ PASSO 4 ]
Criar Conta /       ->  Configurar Perfil   ->  Acessar Menu        ->  Entrar na Crew
Login pwn.college       & Chave SSH             "Crews"                 "Cyberguard"
```

### Passo 1: Cadastro e Autenticação
1. Acesse o portal oficial da plataforma: [pwn.college](https://pwn.college).
2. Clique em **Register** (ou faça Login caso já possua conta via GitHub ou credenciais próprias).
3. **Padrão de Nome de Usuário (Username):** Recomendamos utilizar o formato `nome.sobrenome` ou o seu handle conhecido dentro da Cyberguard para facilitar a identificação pelos mentores.

### Passo 2: Configuração de Perfil e Chave SSH
1. No canto superior direito, clique em seu avatar e vá em **Settings**.
2. Preencha seu nome e certifique-se de que o perfil está visível para a plataforma.
3. Cole sua chave pública SSH no campo **"SSH Public Key"** (consulte a seção de SSH do `DESCRIPTION.md`). Isso habilitará seu acesso direto via terminal local aos containers dos desafios.

### Passo 3: Localizando a Aba de Crews
1. Na barra superior de navegação da plataforma, clique em **Crews** (ou acesse a URL direta `/crews`).
2. Você verá a listagem de equipes registradas e o ranking geral.

### Passo 4: Vinculando-se à Crew Oficial
* **Caso a Crew já esteja criada pelo Capitão:**
  1. Digite `Cyberguard` na barra de pesquisa da página de Crews.
  2. Clique sobre o nome da nossa equipe.
  3. Clique no botão **"Join Crew"** (Entrar na Crew).
  4. Caso seja solicitado o token de convite privado da Cyberguard, insira a chave distribuída no nosso canal interno fechado do Discord/WhatsApp.
* **Instrução para a Liderança (Criação Inicial):**
  1. Se a crew ainda não foi inicializada na instância, o Capitão do time (Giovanni Marin ou Arthur Stellato) deve clicar em **"Create Crew"**.
  2. Nome: `Cyberguard`
  3. Descrição: `Cyberguard Security Team — Pesquisa, Defesa e Operações Ofensivas`
  4. Definir a política de adesão com chave secreta para garantir que apenas membros autorizados ingressem.

---

## 3. Sincronização de Progresso e Placar

* **Registro Automático:** Não é necessária nenhuma ação manual para sincronizar o placar. Toda vez que você resolver um desafio no Dojo `cyberguard-ethical-hacking` e submeter uma flag válida, o sistema da plataforma atualiza instantaneamente o seu total de pontos e soma-o ao placar global da Cyberguard.
* **Transparência de Flags:** O painel da Crew exibe quais módulos foram concluídos por cada membro, criando um mapa de calor (*heatmap*) de competências da equipe.

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
