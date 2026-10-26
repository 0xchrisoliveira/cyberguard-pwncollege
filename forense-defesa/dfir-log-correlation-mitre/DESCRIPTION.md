# Correlação de Logs e Mapeamento MITRE ATT&CK

### 📖 Introdução Teórica

No ecossistema moderno de defesa cibernética (Blue Team e SOC - *Security Operations Center*), a detecção de ameaças não se limita à análise isolada de eventos. Um único login com falha é um ruído corriqueiro de rede; no entanto, cinquenta logins falhados seguidos de um login bem-sucedido via conta de serviço e o agendamento imediato de uma tarefa no Cron representam um incidente crítico de intrusão e movimentação lateral. A capacidade de conectar esses pontos é a essência do **SIEM** e do **Mapeamento Tático MITRE ATT&CK**.

#### A Engenharia de um SIEM (Security Information and Event Management)
O SIEM é a plataforma centralizadora de inteligência defensiva que processa fluxos contínuos de telemetria através de três etapas sequenciais:
1. **Ingestão e Normalização (Parsing):** Coletores (como Elastic Agent, Fluentbit ou Splunk Forwarder) capturam logs brutos de roteadores, firewalls, daemons Linux (`sshd`, `auditd`) e controladores de domínio. Esses dados heterogêneos são normalizados em padrões abertos como o **ECS** (*Elastic Common Schema*) ou o **CIM** (*Common Information Model*), padronizando nomes de campos (ex: `source.ip`, `user.name`, `event.action`).
2. **Motores de Correlação:** Aplicam lógicas temporais e relacionais entre fontes distintas para identificar cadeias complexas de ataque (ex: *Eventos A, B e C ocorrendo dentro de uma janela de 5 minutos com a mesma origem*).
3. **Triagem e Alerta:** Notificam os analistas do SOC com contexto enriquecido e nível de severidade calibrado.

```
+-----------------------------------------------------------------------------------------+
|                                    MITRE ATT&CK MATRIX                                  |
+-------------------+-------------------+------------------------+------------------------+
| INITIAL ACCESS    | PERSISTENCE       | PRIVILEGE ESCALATION   | DEFENSE EVASION        |
+-------------------+-------------------+------------------------+------------------------+
| T1078             | T1053.003         | T1548.001              | T1070.004              |
| Valid Accounts    | Cron / Scheduled  | Setuid / SUID Abuse    | File Deletion          |
| (svc_backup)      | (Retorno furtivo) | (Escalação para root)  | (Apaga vestígios)      |
+-------------------+-------------------+------------------------+------------------------+
```

#### A Taxonomia MITRE ATT&CK
O framework **MITRE ATT&CK** (*Adversarial Tactics, Techniques, and Common Knowledge*) é a base pública global que cataloga os comportamentos reais observados em ataques de adversários e grupos de espionagem cibernética (APTs). Ele organiza as ações em dois níveis indispensáveis:
* **Táticas (Tactics):** O objetivo tático do invasor em cada fase da campanha (*"O que o adversário quer alcançar?"*). Exemplos: *Initial Access*, *Persistence*, *Privilege Escalation*, *Defense Evasion*, *Credential Access*.
* **Técnicas e Subtécnicas (Techniques):** O método técnico específico utilizado para atingir o objetivo (*"Como ele executa a ação?"*). Cada técnica possui um identificador canônico (ex: `T1053.003` para Cron no Linux).

Ao etiquetar cada regra do SIEM com o ID correspondente da matriz ATT&CK, o SOC quantifica com precisão sua cobertura defensiva e identifica pontos cegos na infraestrutura.

---

### 💻 Exemplos Práticos e Sintaxe

A investigação prática de incidentes em ambiente Linux combina ferramentas de correlação de linha de comando com regras estruturadas de detecção:

#### 1. Filtrando Eventos Relevantes no Linux (`auth.log` e `auditd`)
```bash
# Investigar sequências de falhas seguidas de sucessos no serviço SSH:
grep -E "(Failed password|Accepted publickey)" /var/log/auth.log

# Buscar comandos privilegiados registrados pelo framework de auditoria auditd:
ausearch -m EXECVE -ts today

# Gerar resumo estatístico das execuções suspeitas:
aureport -x --summary
```

#### 2. Amostra de Telemetria Correlacionada (`/tmp/<resultado_da_auditoria>.log`)
```text
Oct 26 03:14:10 sec-srv-01 sshd[1204]: Failed password for invalid user admin from 192.168.1.100 port 49152 ssh2
Oct 26 03:14:15 sec-srv-01 sshd[1205]: Failed password for invalid user admin from 192.168.1.100 port 49154 ssh2
Oct 26 03:15:02 sec-srv-01 sshd[1210]: Accepted publickey for svc_backup from 192.168.1.100 port 49160 ssh2: ED25519 SHA256:7uX9a...
Oct 26 03:16:00 sec-srv-01 CRON[1250]: (root) CMD (cp /bin/bash /tmp/.pwned && chmod +s /tmp/.pwned)
Oct 26 03:16:45 sec-srv-01 auditd[900]: type=EXECVE msg=audit(1698300000.120:45): argc=3 a0="/bin/sh" a1="-c" a2="cat /flag"
```

#### 3. Especificação de Regra de Detecção SIEM (Sigma Rule)
```yaml
title: Escalacao de Privilegio via Cron e SUID Apos Login Suspeito
status: production
description: Detecta criacao de binario setuid disparado via tarefa Cron logo apos autenticacao
logsource:
  product: linux
  service: syslog
detection:
  condition: selection_cron and selection_suid
  selection_cron:
    process.name: 'cron'
  selection_suid:
    process.command_line|contains: 'chmod +s'
tags:
  - attack.persistence
  - attack.t1053.003
  - attack.privilege_escalation
  - attack.t1548.001
```

---

### 🎯 Seu Objetivo

Analisar eventos de log de autenticação e auditoria do sistema em busca de indicadores de ataque (*Indicators of Attack*). Seu objetivo é correlacionar a sequência de eventos suspeitos registrados nos logs com a técnica correspondente do framework MITRE ATT&CK, preenchendo o relatório de incidente para comprovar a atribuição da atividade adversária e obter a flag.
