# Correlação de Logs e Mapeamento MITRE ATT&CK

## Briefing da Missão
O Security Operations Center (SOC) da Cyberguard registrou uma anomalia de correlação nos coletores de telemetria do ambiente de produção. O motor SIEM disparou um incidente de severidade alta após processar múltiplos eventos encadeados: sequências repetidas de autenticação rejeitada seguidas de uma sessão aceita via chave pública para a conta de serviço `svc_backup`, acompanhada do agendamento de uma tarefa Cron suspeita e execução privilegiada de comandos.

Como analista de segurança defensiva e inteligência de ameaças, sua missão é examinar o log em `/tmp/cyberguard_auth_audit.log`, compreender a mecânica de correlação do SIEM e classificar o papel da taxonomia MITRE ATT&CK no ecossistema de resposta a incidentes.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

No diagnóstico do ciclo 2026.2 da Cyberguard, **62,5% da equipe errou questões relativas a SIEM e MITRE ATT&CK**, evidenciando a necessidade de consolidar a teoria moderna de Blue Team.

### 1. A Engenharia de um SIEM
Um SIEM (*Security Information and Event Management*) não é apenas um repositório passivo de arquivos de texto. Ele opera um pipeline contínuo de processamento:
1. **Ingestão de Dados:** Agentes (Elastic Agent, Splunk Forwarder, Fluentbit) enviam logs de endpoints, firewalls, servidores web e controladores de domínio.
2. **Normalização (Parsing):** O SIEM converte logs heterogêneos em campos padronizados (ex: `user.name`, `source.ip`, `event.action`).
3. **Regras de Correlação:** Lógicas booleanas e temporais que detectam cadeias de ataque. Por exemplo:
   ```text
   QUANDO (Falhas_de_Login >= 5 em 1 minuto PARA mesmo IP_Origem)
   E DEPOIS (Sucesso_de_Login PARA mesmo IP_Origem em até 5 minutos)
   E DEPOIS (Execução_de_Processo com UID 0 originado por esse usuário)
   ENTÃO DISPARAR: ALERTA CRÍTICO - Compromisso de Conta & Movimentação
   ```

### 2. A Matriz MITRE ATT&CK
O **MITRE ATT&CK** (*Adversarial Tactics, Techniques, and Common Knowledge*) é o mapa de navegação do defensor contemporâneo.

```
+-----------------------------------------------------------------------------------------+
|                                    MITRE ATT&CK MATRIX                                  |
+-------------------+-------------------+------------------------+------------------------+
| INITIAL ACCESS    | PERSISTENCE       | PRIVILEGE ESCALATION   | DEFENSE EVASION        |
+-------------------+-------------------+------------------------+------------------------+
| T1078             | T1053.003         | T1548.001              | T1070.004              |
| Valid Accounts    | Cron / Scheduled  | Setuid / SUID Abuse    | File Deletion          |
| (Contas de serv.) | (Garante retorno) | (Obtém UID 0 / root)   | (Apaga histórico bash) |
+-------------------+-------------------+------------------------+------------------------+
```

* **Táticas (Tactics):** O objetivo tático do invasor naquele momento (O QUÊ).
* **Técnicas (Techniques) & Subtécnicas:** O procedimento técnico específico para alcançar a tática (COMO).
* **Mapeamento Prático:** Cada regra de SIEM é etiquetada com uma técnica (ex: `mitre.technique.id: T1053.003`). Isso permite saber quais táticas estão bem cobertas e quais possuem pontos cegos na organização.

---

## Passo a Passo de Resolução

1. Execute o desafio para iniciar a análise:
   ```bash
   /challenge/dfir-log-correlation-mitre
   ```
2. Examine a amostra de logs gravada em `/tmp/cyberguard_auth_audit.log`:
   ```bash
   cat /tmp/cyberguard_auth_audit.log
   ```
   Observe a linha de execução do Cron abusando de bit SUID (`chmod +s /tmp/.pwned`).
3. Analise as opções fornecidas pelo binário sobre a definição e finalidade do MITRE ATT&CK.
4. Selecione a opção **`2`** (base global de conhecimento de táticas e técnicas adversárias para orientar detecções e SIEM).
5. O debriefing técnico demonstrará as técnicas MITRE correlacionadas e liberará a flag.

---

## Dicas e Comandos de Terminal

- **Filtrando Linhas de Auditoria Relevantes com Grep:**
  ```bash
  grep -E "(CRON|Accepted|EXECVE)" /tmp/cyberguard_auth_audit.log
  ```
- **Pesquisando Técnicas no Portal do MITRE:**
  Acesse a documentação oficial para entender o contexto de cada técnica:
  `https://attack.mitre.org/techniques/T1053/003/`
- **Consultando o Framework Auditd no Linux:**
  ```bash
  sudo aureport -u     # Relatório de execuções por usuário
  sudo ausearch -m USER_AUTH # Busca eventos de autenticação
  ```

---

## Aplicação Prática no Mundo Real

Times maduros de Red Team e Blue Team operam em sinergia através do **Purple Teaming**. O Red Team simula uma técnica do MITRE ATT&CK (como T1053.003 de Cron), e os engenheiros de detecção verificam se os logs do Linux geraram o evento no SIEM e se a regra de correlação disparou o alerta no SOC em menos de 5 minutos. Sem o mapeamento MITRE, a defesa opera de forma reativa e desorganizada.
