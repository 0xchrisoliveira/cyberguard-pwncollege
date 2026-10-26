# 🛡️ Cyberguard Hub: Dojo pwn.college & Trilha de Hacking Ético

Bem-vindo ao repositório central de treinamento em **Hacking Ético e Segurança Ofensiva/Defensiva da Cyberguard**, estruturado para a plataforma **[pwn.college](https://pwn.college)**.

Este projeto foi construído após uma rigorosa avaliação diagnóstica individual dos 8 membros do time (Ciclo 2026.2), identificando as maiores dificuldades e carências técnicas da equipe para desenhar uma jornada de aprendizado progressiva, prática e 100% traduzida e adaptada para **Português do Brasil (PT-BR)**.

---

## 📂 Estrutura do Repositório

```bash
pwncollegeCG/
├── README.md                                  # Visão geral do repositório e índice do projeto
├── dojo.yml                                   # Especificação oficial do Dojo pwn.college (9 módulos, 36 desafios)
├── DESCRIPTION.md                             # Apresentação, regras e guia de uso dos containers pwn.college
├── CREW_CYBERGUARD_SETUP.md                   # Manual de criação/adesão à Crew Cyberguard e trilhas por perfil
│
├── Relatorios/
│   ├── relatorio-individual-2026.2.csv        # Dados brutos das 20 questões e notas dos 8 membros
│   └── DIAGNOSTICO_DESEMPENHO_CYBERGUARD.md   # Relatório analítico profundo com métricas, causas-raiz e personas
│
└── conteudo_ptbr/                             # Biblioteca didática completa em PT-BR para nivelamento
    ├── 01_linux_essentials_permissoes.md      # FHS, cálculo octal (chmod 640), SUID e comandos essenciais
    ├── 02_redes_tcp_handshake_portas.md       # Three-Way Handshake TCP, flags, portas, netcat, tcpdump e nmap
    ├── 03_engenharia_reversa_x86_rip_gdb.md   # Registradores x86_64, RIP/EIP, Ghidra, GDB/GEF e análise estática vs dinâmica
    ├── 04_exploracao_binaria_buffer_overflow.md# Stack frame, strcpy vulnerável, cálculo de offset e exploit em pwntools
    ├── 05_web_pentest_sqli_ssrf.md           # SQL Injection, Prepared Statements (por que blacklist falha) e SSRF
    ├── 06_forense_volatilidade_memoria.md     # RFC 3227 (Ordem de Volatilidade: RAM em 1º), Cadeia de Custódia e SIEM/MITRE
    ├── 07_active_directory_kerberoasting.md  # Domain Controllers, protocolo Kerberos e ataque de Kerberoasting
    └── 08_criptografia_e_ctf_basics.md       # Base64 vs Hashes, por que bcrypt/Argon2 para senhas e dicas de CTF
```

---

## 📊 1. Síntese do Diagnóstico da Equipe (Ciclo 2026.2)

A partir da avaliação diagnóstica individual dos 8 membros da Cyberguard, apurou-se uma taxa média de acerto global de **39,38% (7,88 / 20 questões)**, revelando uma nítida assimetria na equipe:

```
[ RANKING GERAL DE ACERTOS ]
1. Giovanni Mastelaro Marin: 17/20 (85%) ── [Avançado]
2. Arthur Gabriel Teotonio:  14/20 (70%) ── [Avançado]
3. Silvio Henrique Mendes:   14/20 (70%) ── [Avançado]
4. Lauren Marçulo:            9/20 (45%) ── [Intermediário / Vocação Blue Team]
5. Joao Vitor Cordeiro:       4/20 (20%) ── [Base / Onboarding]
6. Ismael Rodrigues G. Neto:  2/20 (10%) ── [Base / Onboarding]
7. Matheus Hirose Rodrigues:  2/20 (10%) ── [Base / Onboarding]
8. Luiz Paulo Tonini Guerra:  1/20 ( 5%) ── [Base / Onboarding]
```

### Principais Lacunas Identificadas (Média de 0 a 100):
1. **Engenharia Reversa e Binários (5.0 / 100 - Crítico Absoluto):**
   * **100% dos membros (8 de 8)** erraram a função do registrador **EIP/RIP** em x86/x64 (a maioria confundiu com flags ou admitiu desconhecimento).
   * **87,5% dos membros (7 de 8)** não souberam diferenciar **análise estática de dinâmica**.
2. **Forense Digital e Resposta a Incidentes (12.5 / 100 - Crítico):**
   * **87,5% dos membros (7 de 8)** erraram a **Ordem de Volatilidade (RFC 3227)**, priorizando logs em disco antes da memória RAM.
   * **75% dos membros (6 de 8)** desconheciam o conceito formal de **Cadeia de Custódia**.
3. **Active Directory & Infraestrutura (22.5 / 100):**
   * **75% dos membros (6 de 8)** erraram o funcionamento do ataque de **Kerberoasting**.
   * **62,5% (5 de 8)** não souberam definir o papel do **Controlador de Domínio (DC)**.
4. **Fundamentos (Linux & Redes - 37.1 / 100):**
   * **62,5% (5 de 8)** erraram a permissão octal **`chmod 640`** (acreditando que liberava acesso geral ou desconheciam).
   * **62,5% (5 de 8)** erraram a ordem do **Three-Way Handshake TCP** (`SYN` -> `SYN-ACK` -> `ACK`).
5. **O Paradoxo de Programação (77.5 / 100):**
   * **100% da equipe (8 de 8)** acertou o motivo pelo qual a função `strcpy` em C é perigosa (falta de checagem de limites de buffer), indicando que possuem a base conceitual de código perigoso, mas sem a vivência prática de como isso se traduz no mapa de memória da CPU (Stack Overflow e controle do RIP).

> 📄 *Para o relatório completo com detalhamento estatístico, causa-raiz de cada questão e dossiê individual por membro, leia: [`Relatorios/DIAGNOSTICO_DESEMPENHO_CYBERGUARD.md`](Relatorios/DIAGNOSTICO_DESEMPENHO_CYBERGUARD.md).*

---

## 🎯 2. Arquitetura da Trilha no pwn.college (`dojo.yml`)

O Dojo **`cyberguard-ethical-hacking`** foi organizado de forma progressiva em **9 módulos temáticos (36 desafios)**:

| Módulo | Foco Temático | Desafios Chave |
| :--- | :--- | :--- |
| **1. Fundamentos Linux** | Navegação, I/O, pipes, variáveis e cálculo octal | Permissões 640/755, bit SUID e escalação |
| **2. Redes & Protocolos** | Pilha TCP/IP, Handshake TCP, portas e tráfego | Dissecando Handshake SYN/ACK, Netcat e nmap |
| **3. Scripting Ofensivo** | Automação com Bash e Python | Sockets, pwntools básico, strace/ltrace |
| **4. Web Pentest (OWASP)** | OWASP Top 10, SQLi e SSRF | Quebra de Blacklist vs Prepared Statements, SSRF |
| **5. Criptografia & CTF** | Encodings (Base64), Hashes lentos (Argon2/bcrypt) | Decodificando Base64, quebra de hashes, salt |
| **6. Forense & Blue Team** | Ordem de Volatilidade (RAM), Custódia e SIEM | Análise de dump de memória, Volatility, MITRE |
| **7. Active Directory** | Domain Controllers, Kerberos e escalação AD | Kerberoasting prático e análise com BloodHound |
| **8. Engenharia Reversa** | Arquitetura x86_64, RIP/EIP, Ghidra e GDB | Identificando RIP no GDB, estática vs dinâmica |
| **9. Exploração Binária** | Buffer Overflow na Stack, estouro de strcpy | Offset com cyclic, RIP hijacking e shellcode |

---

## 👥 3. Configuração da Crew no pwn.college

O arquivo [`CREW_CYBERGUARD_SETUP.md`](CREW_CYBERGUARD_SETUP.md) estabelece como a equipe deve operar unida no placar da plataforma. As trilhas recomendadas por grupo são:

1. **Cohort 1 — Fundacional & Onboarding Técnico**
   * *Membros:* Luiz Paulo Tonini, Ismael Rodrigues, Matheus Hirose, Joao Vitor Cordeiro.
   * *Objetivo:* Construir fluência diária no Linux (Módulo 1), redes (Módulo 2) e scripts básicos em Python (Módulo 3) antes de tentar desafios avançados.
2. **Cohort 2 — Intermediário & Visão Híbrida**
   * *Membro:* Lauren Marçulo.
   * *Objetivo:* Alavancar sua alta proficiência defensiva (Blue Team/SIEM) desenvolvendo visão adversária ofensiva (Web Pentest, Redes e Reversa x86 básica).
3. **Cohort 3 — Veteranos & Mentores de Alto Nível**
   * *Membros:* Giovanni Marin, Arthur Stellato, Silvio Santos.
   * *Objetivo:* Focar prioritariamente nos Módulos 6 (Forense/RAM), 7 (Active Directory) e 8/9 (Engenharia Reversa x86, Registrador RIP e Exploração Binária), além de conduzir sessões semanais de pareamento técnico com os novatos.

---

## 🚀 Como Começar

1. **Para os Membros:**
   * Leia [`DESCRIPTION.md`](DESCRIPTION.md) para entender o ambiente de laboratório e as regras.
   * Siga [`CREW_CYBERGUARD_SETUP.md`](CREW_CYBERGUARD_SETUP.md) para se cadastrar no pwn.college e ingressar na Crew **Cyberguard**.
   * Estude os guias teóricos em [`conteudo_ptbr/`](conteudo_ptbr/) conforme avança pelos módulos do Dojo.
2. **Para a Liderança Técnica:**
   * Analise o relatório em [`Relatorios/DIAGNOSTICO_DESEMPENHO_CYBERGUARD.md`](Relatorios/DIAGNOSTICO_DESEMPENHO_CYBERGUARD.md).
   * Importe o [`dojo.yml`](dojo.yml) na plataforma pwn.college para ativar a trilha para o time.
