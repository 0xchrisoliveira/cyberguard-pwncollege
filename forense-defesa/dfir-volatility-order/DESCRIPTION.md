# Triage Emergencial: Respeitando a RFC 3227

## Briefing da Missão
O Centro de Operações de Segurança (SOC) da Cyberguard emitiu um alerta de intrusão crítica: o servidor de banco de dados `sec-srv-01` está estabelecendo conexões anômalas para um endereço IP desconhecido na porta TCP 4444. Indicadores apontam para um túnel reverso ativo e atividade de um agente malicioso operando em tempo real.

Sua missão como operador do CSIRT (Computer Security Incident Response Team) é intervir no sistema comprometido e determinar a prioridade estrita de aquisição de evidências, garantindo que nenhum dado perecível seja destruído por reinicialização, ação do atacante ou falta de energia.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

No diagnóstico do ciclo 2026.2 da Cyberguard, **87,5% da equipe errou a ordem de prioridade na coleta de evidências forenses**, tendendo a priorizar arquivos estáticos gravados em disco rígido ou logs arquivados.

Essa conduta representa um dos erros mais fatais na resposta a incidentes. A norma internacional **RFC 3227** (*Guidelines for Evidence Collection and Archiving*) determina que a aquisição forense deve seguir rigorosamente a **Ordem de Volatilidade**: colete primeiro os artefatos que desaparecem mais rápido.

### Diagrama da Ordem de Volatilidade (RFC 3227)

```
[ MAIS VOLÁTIL - Desaparece em microssegundos/minutos ]
        ▲
        │  1. Registradores da CPU e Memória Cache
        │  2. Tabelas de Roteamento, Cache ARP, Tabela de Processos, Conexões TCP
        │  3. Memória Principal (Memória RAM Física)
        │  4. Sistemas de Arquivos Temporários (/tmp, /dev/shm, swap, pagefile)
        │  5. Discos Rígidos, SSDs e Logs Locais em Disco (/var/log)
        │  6. Logs em Servidores Remotos (SIEM, syslog-ng) e Capturas de Rede
        │  7. Mídias Físicas de Arquivamento (Fitas magnéticas, backups a frio)
        ▼
[ MENOS VOLÁTIL - Persiste por meses ou anos sem energia ]
```

### Por que a Memória RAM Tem Prioridade Absoluta Sobre o Disco?

| Aspecto | Memória RAM (Volátil) | Disco Rígido / Logs (Não-Volátil) |
| :--- | :--- | :--- |
| **Persistência** | Destruída instantaneamente se o servidor perder energia ou sofrer reboot. | Permanece gravada mesmo com o servidor completamente desligado. |
| **Fileless Malware** | Executa puramente na RAM (*Process Hollowing*, payloads reflexivos) sem gravar arquivo em disco. | Invisível no sistema de arquivos tradicional. |
| **Credenciais** | Senhas em texto claro, tokens de sessão, hashes NTLM e tickets Kerberos residem na memória. | Senhas raramente são registradas em texto puro nos logs. |
| **Chaves de Criptografia**| Chaves simétricas (AES-256) de ransomware ou tráfego TLS encontram-se em buffers de memória. | Uma vez reiniciada a máquina, a chave é irrecuperável. |
| **Comando e Controle (C2)**| Sockets de rede abertos e descritores de arquivo de túneis reversos residem na memória do kernel. | Podem não constar em logs caso o atacante tenha desativado o log local. |

---

## Passo a Passo de Resolução

1. Execute o binário do desafio no terminal:
   ```bash
   /challenge/dfir-volatility-order
   ```
2. Caso precise de uma revisão conceitual completa dos princípios da RFC 3227, passe o argumento `--help`:
   ```bash
   /challenge/dfir-volatility-order --help
   ```
3. O desafio apresentará as quatro fontes de evidência disponíveis:
   - A) Disco Rígido e Logs salvos (`/var/log/auth.log`)
   - B) Memória RAM física volátil (processos vivos, chaves, sockets)
   - C) Fitas de Backup externo e imagens de restauração
   - D) Arquivos de configuração do sistema em `/etc/`
4. Selecione a opção **`B`** (ou passe `B` diretamente como argumento).
5. O debriefing técnico será emitido e a flag de validação será impressa.

---

## Dicas e Comandos de Terminal

- **Módulo LiME (Linux Memory Extractor):**
  Para extrair a RAM em um host Linux sem contaminar a memória do usuário:
  ```bash
  insmod lime-$(uname -r).ko "path=/tmp/ram_dump.lime format=raw"
  ```
- **Aquisição em Windows via WinPmem:**
  ```cmd
  winpmem.exe -o mem_corporativa.raw
  ```
- **Garantia de Integridade (Cadeia de Custódia):**
  Gere imediatamente o hash criptográfico do dump para validação pericial:
  ```bash
  sha256sum /tmp/ram_dump.lime | tee ram_dump.lime.sha256
  ```

---

## Aplicação Prática no Mundo Real

Em incidentes modernos de ransomware corporativo, os atacantes costumam injetar scripts maliciosos em processos legítimos (`powershell.exe`, `svchost.exe` ou `systemd`) e configurar comandos de destruição imediata caso percebam a intervenção do administrador. Se a equipe de resposta simplesmente desligar o servidor ou rodar scripts de coleta de disco primeiro, a chave de descriptografia mantida em RAM será destruída, inviabilizando a recuperação dos dados sem o pagamento de resgate. Respeitar a RFC 3227 é a linha divisória entre conter o incidente com sucesso ou amargar a destruição de todo o ambiente.
