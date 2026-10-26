# Mapeamento de Rotas de Ataque com BloodHound (AD BloodHound Path)

## Briefing da Missão
* **Codinome:** Operação Path to DA
* **Classificação:** Confidencial / Treinamento de Red Team & Análise de Grafos em AD
* **Alvo:** Grafo de Controle de Acesso e Permissões Transitivas do Active Directory
* **Objetivo:** Analisar um grafo de permissões gerado pelo BloodHound/SharpHound, rastrear a cadeia transitiva de direitos (`MemberOf` -> `GenericAll` -> `WriteDacl`) e identificar o nó de usuário inicial capaz de escalar privilégios até o grupo `Domain Admins`.

Em redes corporativas modernas, administradores costumam conceder permissões pontuais ao longo dos anos para resolver demandas emergenciais. Com o tempo, a sobreposição dessas permissões cria pontes invisíveis entre contas sem privilégios e o grupo de administradores do domínio, sem que nenhum patch ou antivírus seja violado.

---

## Teoria Aprofundada

### 1. A Filosofia do BloodHound: Atacantes Pensam em Grafos
John Lambert (arquiteto de segurança da Microsoft) cunhou a célebre frase:
> *"Defenders think in lists. Attackers think in graphs. As long as this is true, attackers win."*

Enquanto a defesa verifica listas de checagem isoladas (ex: *"Quem está no grupo Domain Admins?"*), o invasor busca conexões indiretas:

```
[ Usuário Bob ] 
       │
   (MemberOf)
       ▼
[ Grupo HelpDesk ] ──(GenericAll)──> [ Conta: srv_backup ] ──(WriteDacl)──> [ Grupo Domain Admins ]
```

Nenhum desses nós é diretamente um Administrador do Domínio. Porém, quando encadeados, conferem controle absoluto sobre toda a infraestrutura corporativa.

### 2. Dicionário das Principais Arestas (Edges) no BloodHound

| Tipo de Aresta (Edge) | Tradução Prática do Privilégio | Impacto Ofensivo de Abuso |
| :--- | :--- | :--- |
| **`MemberOf`** | O nó pertence ao grupo de destino | Herda todos os privilégios daquele grupo corporativo |
| **`GenericAll`** | Controle total e irrestrito sobre o objeto | Redefinir senha, adicionar atributos SPN, alterar dados |
| **`GenericWrite`** | Permissão de escrita em qualquer atributo | Modificar scripts de logon ou atributos de delegação |
| **`WriteDacl`** | Direito de reescrever a DACL do objeto | Conceder a si mesmo `GenericAll` sobre a vítima |
| **`WriteOwner`** | Tomar posse proprietária do objeto | Tornar-se dono e atribuir permissões totais |
| **`ForceChangePassword`**| Redefinição forçada de credencial | Trocar a senha do alvo sem conhecer a senha atual |
| **`AdminTo`** | Administrador Local da máquina | Executar dump de memória do processo LSASS |

### 3. A Mecânica da Escalação Passo a Passo:
1. **Passo 1:** Bob é autenticado como usuário padrão e pertence ao grupo de suporte `HelpDesk`.
2. **Passo 2:** O grupo `HelpDesk` recebeu `GenericAll` sobre a conta de serviço `srv_backup` para manutenção de rotinas de backup. Bob usa essa permissão para alterar a senha da conta `srv_backup`.
3. **Passo 3:** A conta `srv_backup` possui `WriteDacl` sobre o grupo `Domain Admins`. Bob autentica-se como `srv_backup` e altera a lista de permissões do grupo `Domain Admins`, concedendo a si mesmo o direito de adicionar novos membros.
4. **Passo 4:** Bob adiciona seu próprio usuário original (`bob`) ao grupo `Domain Admins`. Escalação de privilégios concluída (*Domain Compromise*).

---

## Passo a Passo de Resolução

1. Execute o desafio para visualizar o grafo de permissões:
   ```bash
   /challenge/ad-bloodhound-path
   ```
2. Caso deseje consultar a fundamentação técnica sobre teoria dos grafos e BloodHound:
   ```bash
   /challenge/ad-bloodhound-path --help
   ```
3. O desafio apresentará a rota:
   ```text
   [User: bob] -> MemberOf -> [Group: HelpDesk] -> GenericAll -> [User: srv_backup] -> WriteDacl -> [Group: Domain Admins]
   ```
4. Identifique o usuário inicial que desencadeia a cadeia de ataque:
   ```text
   bob
   ```
5. Você também pode disparar a execução passando o nome do usuário como argumento:
   ```bash
   /challenge/ad-bloodhound-path bob
   ```
6. O sistema validará a análise da rota, exibirá o debriefing de arquitetura de segurança e liberará a flag.

---

## Dicas e Comandos Práticos

* **Coleta de Dados em Campo com SharpHound (Windows):**
  ```powershell
  # Coleta todos os dados de AD, sessões e ACLs compactados em arquivo zip:
  .\SharpHound.exe --CollectionMethods All --Domain corp.cyberguard.local --ZipFileName ad_dump.zip
  ```
* **Consultas Úteis em Linguagem Cypher no Neo4j (Interface do BloodHound):**
  ```cypher
  // Encontrar o menor caminho de qualquer usuário não privilegiado até Domain Admins:
  MATCH (u:User {admincount: 0}), (g:Group {name: "DOMAIN ADMINS@CORP.LOCAL"}),
        p = shortestPath((u)-[*1..]->(g))
  RETURN p
  ```
* **Identificando Usuários com Sessão Ativa em Servidores Críticos:**
  A aresta `HasSession` indica que o usuário possui credenciais na memória RAM do computador (LSASS), permitindo roubo de token (*token stealing*) ou *mimikatz*.

---

## Aplicação Prática no Mundo Real
O BloodHound é a principal ferramenta utilizada por equipes de Red Team e auditores de segurança para demonstrar aos conselhos de administração como uma falha de configuração aparentemente menor (como um usuário de HelpDesk com permissões residuais de anos atrás) permite a tomada de controle total da empresa em menos de 15 minutos.

A defesa contra esses ataques baseia-se na implementação do **Modelo de Camadas Administrativas (Enterprise Access Tiering)**:
* **Tier 0:** Domain Controllers, contas de administradores de domínio e servidores de identidade. Nenhuma conta de Tier inferior pode ter qualquer direito de escrita sobre objetos Tier 0.
* **Tier 1:** Servidores de aplicação corporativos.
* **Tier 2:** Estações de trabalho de usuários comuns.
