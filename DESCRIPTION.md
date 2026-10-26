# Cyberguard: Trilha Completa de Hacking Ético
## Dojo Oficial de Capacitação e Alta Performance Técnica — Cyberguard

---

## 1. Apresentação do Dojo

Bem-vindo à trilha oficial de capacitação em segurança ofensiva e defensiva da **Cyberguard** hospedada na infraestrutura do **pwn.college**!

Este ambiente foi projetado especificamente para transformar teoria em prática rigorosa de laboratório. No mundo da cibersegurança, conhecer conceitos em livros ou apresentações de slides é insuficiente quando confrontado com um incidente real ou uma superfície de ataque viva. O objetivo supremo deste dojo é conceder aos membros da Cyberguard o domínio irrestrito da máquina: desde as chamadas de sistema no kernel Linux até as instruções assembly executadas pelo silício da CPU.

### A Filosofia Educacional pwn.college
O **pwn.college** é uma plataforma educacional de ponta desenvolvida por pesquisadores da Arizona State University (ASU) e liderada por Yan Shoshitaishvili (`zardus`). Diferente de plataformas tradicionais baseadas em caça de pistas arbitrárias ("guessing CTFs"), o pwn.college fundamenta-se em:
* **Ambiente Real Baseado em Containers e Kernel Linux:** Cada desafio roda em um container dedicado e isolado, garantindo um sistema operacional completo para exploração.
* **Aprendizagem Baseada em Resolução Prática de Problemas (Problem-Based Learning):** Não há atalhos. Para avançar, você precisa compreender exatamente como a memória, a rede e o software interagem.
* **Transparência Pedagógica:** As ferramentas utilizadas aqui são as mesmas empregadas por analistas de Red Team, engenheiros reversos e peritos de Blue Team em operações corporativas reais (GDB, Ghidra, Pwntools, Wireshark, Volatility, Strace).

---

## 2. Objetivos de Aprendizagem

Ao concluir esta trilha, o membro da Cyberguard terá desenvolvido as seguintes competências essenciais:

1. **Fluência Operacional em Linux:** Operar ambientes Unix exclusivamente por linha de comando, manipulando fluxos de dados com pipes, redirecionamentos e compreendendo a fundo o modelo de permissões octais e bits especiais (`SUID`/`SGID`).
2. **Visão de Tráfego e Redes:** Compreender cada etapa do ciclo de vida das conexões de rede, dissecando pacotes TCP/IP, interpretando o Three-Way Handshake (`SYN` -> `SYN-ACK` -> `ACK`) e utilizando ferramentas de captura e análise de tráfego.
3. **Automação de Tarefas e Engenharia de Exploits:** Desenvolver ferramentas e scripts em Python e Bash para dialogar com serviços via sockets e interagir com processos binários usando `pwntools`.
4. **Pensamento Crítico em Segurança Web (OWASP):** Identificar, explorar e implementar defesas definitivas contra falhas clássicas, eliminando vícios como filtragens por lista negra (*blacklists*) e adotando consultas parametrizadas (*prepared statements*) contra SQL Injection, além de compreender ataques de SSRF.
5. **Diferenciação Criptográfica:** Reconhecer instantaneamente a diferença entre codificações (Base64, Hexadecimal), hashes de integridade (SHA-256) e funções lentas de derivação de chave com salt (Bcrypt, Argon2) para proteção de senhas.
6. **Resposta a Incidentes e Preservação de Evidências (DFIR):** Aplicar com rigor a Ordem de Volatilidade (RFC 3227), garantindo que dados voláteis na memória RAM sejam adquiridos antes da manipulação de mídias de disco estáticas, resguardando a Cadeia de Custódia.
7. **Engenharia Reversa de Baixo Nível:** Ler e interpretar código de máquina x86_64, diferenciando análise estática de dinâmica, e dominando o mapa de registradores da CPU — em especial a função crítica do ponteiro de instruções (`RIP`/`EIP`).
8. **Corrupção de Memória e Exploração de Binários:** Identificar funções vulneráveis em C (`strcpy`, `gets`), desenhar o layout de um quadro de pilha (*stack frame*), calcular offsets de estouro de buffer e sequestrar o ponteiro de execução para obter shells privilegiados.

---

## 3. Código de Ética e Regras de Conduta

A prática de hacking ético exige responsabilidade moral e respeito incondicional às leis e às pessoas. O acesso a este dojo pressupõe a adesão aos seguintes termos:

* **Princípio da Autorização Estrita:** Os conhecimentos, técnicas e ferramentas demonstrados neste dojo devem ser utilizados **exclusivamente** dentro deste ambiente de laboratório ou em sistemas onde haja autorização explícita e formal por escrito (pentests contratados, programas de bug bounty com escopo definido).
* **Integridade Acadêmica:** 
  * O aprendizado é o produto final. É terminantemente proibido compartilhar flags diretamente com colegas ou publicar soluções passo a passo ("walkthroughs sem contexto") que anulem o esforço investigativo dos outros membros.
  * O compartilhamento de **métodos, explicações conceituais e sessões de estudo colaborativo** é ativamente encorajado e louvável. Se um colega estiver travado, ajude-o a formular a pergunta certa, aponte a documentação ou ensine a usar a ferramenta de inspeção, mas **nunca envie a flag pronta**.
* **Proteção e Respeito à Infraestrutura:** Não tente atacar a infraestrutura central do pwn.college, outros containers de colegas ou serviços que não façam parte expressa do desafio em execução. Pratique ataques de negação de serviço (DoS) apenas se expressamente requisitado pelo enunciado do laboratório.

---

## 4. Pré-requisitos & Quem Pode Participar

**Não há pré-requisito eliminatório para iniciar o dojo.**

Mesmo que você nunca tenha aberto um terminal Linux ou escrito uma linha de código em Python na vida, a trilha foi estruturada de forma progressiva. Metade da nossa equipe iniciou no nível fundacional.

* **Para Iniciantes Absolutos:** O Módulo 1 (`fundamentos-linux`) e o Módulo 2 (`redes-protocolos`) foram construídos para serem o seu ponto de partida guiado. Tenha paciência, leia atentamente as descrições dos comandos e faça perguntas nos nossos canais internos.
* **Para Membros Intermediários e Avançados:** O dojo desafiará você nos módulos de baixo nível (Engenharia Reversa, Exploração de Binários e Análise Forense de Memória). Utilize seu tempo livre para atuar como mentor dos ingressantes.

---

## 5. Como Operar o Ambiente do pwn.college

O pwn.college fornece uma infraestrutura baseada na nuvem onde você pode resolver os desafios através de três interfaces complementares:

```
                           +-------------------------------------+
                           |        PLATAFORMA PWN.COLLEGE       |
                           |   https://pwn.college/cyberguard    |
                           +-------------------------------------+
                                              |
                +-----------------------------+-----------------------------+
                |                             |                             |
                v                             v                             v
     +--------------------+        +--------------------+        +--------------------+
     |    WEB TERMINAL    |        |   VS CODE BROWSER  |        |     ACESSO SSH     |
     | (Interface Direta) |        |   (Code-Server)    |        | (Terminal Nativo)  |
     +--------------------+        +--------------------+        +--------------------+
```

### 5.1 Iniciando um Desafio
1. Navegue até o módulo desejado na interface web do dojo.
2. Clique no desafio que pretende resolver.
3. Clique no botão **"Start Challenge"** ou **"Launch"** para que a plataforma provisione o seu container Linux exclusivo.
4. Seu diretório `/home/hacker` é persistente: scripts, anotações e ferramentas que você salvar nele permanecerão disponíveis mesmo após reiniciar instâncias. O restante do sistema operacional é recriado de forma limpa a cada inicialização.

### 5.2 Interface 1: Web Terminal
* Acessível diretamente pelo navegador na aba do desafio.
* Ideal para testes rápidos, navegação simples de arquivos e execução de utilitários de sistema.
* Não requer nenhuma configuração em sua máquina local.

### 5.3 Interface 2: VS Code no Navegador (Code-Server)
* O pwn.college integra uma versão completa do Visual Studio Code executada diretamente no navegador.
* Para abrir, clique na aba **"Workspace"** ou **"Desktop/Code"** do seu container ativo.
* **Benefícios:**
  * Editor de código com realce de sintaxe (*syntax highlighting*) para Python, C e Bash.
  * Explorador de arquivos gráfico para criar e organizar seus scripts em `/home/hacker`.
  * Terminais integrados com suporte a abas e divisão de tela.

### 5.4 Interface 3: Conexão Remota via SSH (Recomendado para Usuários Avançados)
Trabalhar a partir do seu próprio terminal local (Linux, macOS ou WSL2 no Windows) oferece a máxima produtividade e permite utilizar seus próprios atalhos, dotfiles e ferramentas visuais instaladas na sua máquina (como o Ghidra local).

#### Passo a Passo de Configuração do SSH:
1. Gere um par de chaves SSH em sua máquina caso ainda não possua:
   ```bash
   ssh-keygen -t ed25519 -C "seu-email@cyberguard.br"
   ```
2. Visualize e copie sua chave pública:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
3. No pwn.college, acerte em seu **Perfil de Usuário (Settings)**, localize o campo **"SSH Public Key"** e cole o conteúdo copiado. Salve a configuração.
4. Com um desafio iniciado na plataforma, conecte-se via terminal com o comando:
   ```bash
   ssh -i ~/.ssh/id_ed25519 hacker@dojo.pwn.college
   ```
   *(Substitua o host caso utilize uma instância interna do servidor).*

#### Redirecionamento de Portas (SSH Port Forwarding):
Quando estiver resolvendo desafios que possuem interfaces web locais ou serviços de rede internos, você pode encaminhar a porta do container diretamente para o seu navegador local:
```bash
ssh -i ~/.ssh/id_ed25519 -L 8080:127.0.0.1:8080 hacker@dojo.pwn.college
```
Dessa forma, acessar `http://localhost:8080` na sua máquina local carregará a aplicação em execução dentro do container!

---

## 6. O Sistema de Privilégios e a Captura da Flag (`/flag`)

Todo desafio no pwn.college tem um objetivo central: **obter a flag secreta**.

### Como a Flag é Protegida?
* O arquivo com o segredo fica localizado na raiz do sistema de arquivos: `/flag`.
* As permissões padrão de `/flag` são restritivas: `chmod 400 /flag` pertencente ao usuário `root`.
* Você está conectado como o usuário não privilegiado `hacker`. Se tentar ler diretamente com `cat /flag`, o sistema operacional retornará `Permission denied`.
* Em cada desafio, existe um binário alvo (geralmente localizado em `/challenge/run` ou com o nome do desafio). Esse executável possui o bit **SUID** ativo ou roda com privilégios de root delegados pelo container.
* O seu objetivo técnico é interagir corretamente com o binário, passar pelos desafios lógicos ou explorar suas vulnerabilidades de software para que **ele** leia o conteúdo de `/flag` e exiba na sua tela, ou permita que você spawne um shell de root!

### Formato da Flag
As flags do pwn.college possuem o padrão:
```
pwn.college{exemplo_hash_ou_segredo_hexadecimal_aqui}
```
Ao encontrar a string, copie-a e submeta no campo de texto do desafio na plataforma web para que seu progresso seja registrado imediatamente e pontuado no placar da equipe!

---

## 7. Dicas de Ouro para o Sucesso

1. **Faça Anotações Pessoais (Keep a Logbook):** Crie um repositório pessoal ou pasta de notas (Markdown/Obsidian/Notion) documentando cada comando novo aprendido e o raciocínio por trás de cada desafio concluído. O ato de escrever consolida a memória de longo prazo.
2. **Não Pule Etapas:** A tentação de pular direto para exploração de binários sem dominar o terminal ou a arquitetura de registradores leva à frustração. A trilha foi desenhada como uma escada: cada degrau apoia o próximo.
3. **Use as Ferramentas de Inspeção:** Antes de tentar adivinhar o comportamento de um binário, use `file`, `strings`, `ltrace`, `strace` e `gdb`. O computador nunca mente; ele apenas faz exatamente o que o código manda.
4. **Celebre os Pequenos Passos:** Entender uma única instrução assembly ou decodificar seu primeiro pacote no Wireshark é uma grande vitória. O aprendizado em segurança é cumulativo.

*Boa jornada, operador Cyberguard. A fortaleza espera por você!*
