# Mapeamento de Rotas de Ataque com BloodHound

### 📖 Introdução Teórica
Nas palavras clássicas de John Lambert (arquiteto distinto de segurança da Microsoft):
> *"Defenders think in lists. Attackers think in graphs. As long as this is true, attackers win."*

Em redes empresariais tradicionais, as equipes de defesa tendem a auditar listas de controle estáticas e isoladas (por exemplo: *"Quem são os usuários listados no grupo Domain Admins?"*). Se essa lista contiver apenas três contas conhecidas, o administrador assume falsamente que o domínio está seguro.
No entanto, invasores pensam em **grafos de relacionamentos e permissões transitivas**.

**Como Permissões Transitivas Criam Caminhos Invisíveis de Escalação:**
Ao longo de anos de operação de TI em uma empresa, centenas de permissões pontuais e delegações são concedidas para atender demandas temporárias (chamados de suporte, integrações de sistemas, scripts legados). Quando essas permissões individuais se encadeiam, elas formam pontes que permitem a qualquer colaborador comum assumir o controle total do domínio (*Domain Compromise Path*).

**O Ecossistema BloodHound:**
O **BloodHound** é uma ferramenta de código aberto que utiliza a teoria dos grafos para mapear relações ocultas de controle de acesso dentro de um Active Directory:
1. **Coletores / Ingestors (SharpHound / BloodHound.py):** Executam milhares de consultas LDAP, chamadas MSRPC/SAMR e enumerações de sessões SMB na rede, gerando arquivos JSON com todos os nós e permissões.
2. **Banco de Dados em Grafo (Neo4j):** Modela os dados em nós (usuários, grupos, computadores, OUs) e arestas direcionadas (*edges*, representando privilégios e associações).
3. **Interface Visual (BloodHound GUI):** Permite calcular rotas mais curtas (*shortest paths*) até grupos de alto privilégio como `Domain Admins`.

**Dicionário das Principais Arestas (Edges) no BloodHound:**
* **`MemberOf`:** O nó pertence a um grupo e herda todos os direitos daquele grupo.
* **`GenericAll`:** Controle total e irrestrito sobre o objeto de destino (permite redefinir a senha do usuário, modificar seus atributos ou adicioná-lo a novos grupos).
* **`WriteDacl`:** Direito de reescrever a lista de controle de acesso discricionário (DACL) do objeto alvo, permitindo ao atacante conceder a si próprio permissão de `GenericAll`.
* **`WriteOwner`:** Permite tomar posse da propriedade do objeto e subsequentemente alterar suas permissões.
* **`ForceChangePassword`:** Permite trocar a senha da conta de destino sem precisar conhecer a senha atual.
* **`AdminTo`:** Usuário tem privilégio de Administrador Local sobre a estação ou servidor.

**Análise da Cadeia de Ataque deste Desafio:**
Considere a rota extraída pelo BloodHound:
```
[User: bob] ──(MemberOf)──> [Group: HelpDesk] ──(GenericAll)──> [User: srv_backup] ──(WriteDacl)──> [Group: Domain Admins]
```
1. O usuário comum **`bob`** pertence ao grupo **`HelpDesk`**.
2. O grupo `HelpDesk` possui permissão total (**`GenericAll`**) sobre a conta de serviço **`srv_backup`**. Bob utiliza esse direito para redefinir a senha de `srv_backup`.
3. A conta `srv_backup` possui a permissão **`WriteDacl`** sobre o grupo restrito **`Domain Admins`**. Bob utiliza a conta `srv_backup` para alterar a lista de acesso do grupo e adicionar seu próprio usuário (`bob`) aos administradores do domínio.
4. Escalação concluída com sucesso: o ponto de partida de toda a cadeia é o usuário **`bob`**.

### 💻 Exemplos Práticos e Sintaxe
1. **Coleta de Dados em Campo com SharpHound (Windows PowerShell):**
   ```powershell
   .\SharpHound.exe --CollectionMethods All --Domain corp.cyberguard.local --ZipFileName ad_coleta.zip
   ```

2. **Coleta de Dados no Linux com `bloodhound-python`:**
   ```bash
   bloodhound-python -u bob -p "Senha123" -d corp.cyberguard.local \
     -dc dc01.corp.cyberguard.local -c All --zip
   ```

3. **Consultas em Linguagem Cypher no Neo4j (Interface do BloodHound):**
   * *Encontrar o caminho mais curto de qualquer usuário comum até Domain Admins:*
     ```cypher
     MATCH (u:User {admincount: 0}), (g:Group {name: "DOMAIN ADMINS@CORP.LOCAL"}),
           p = shortestPath((u)-[*1..]->(g))
     RETURN p
     ```
   * *Localizar todos os membros de grupos com permissão WriteDacl em Domain Admins:*
     ```cypher
     MATCH p=(m)-[:WriteDacl]->(g:Group {name: "DOMAIN ADMINS@CORP.LOCAL"})
     RETURN p
     ```

4. **Defesa Estrutural: Modelo de Camadas Administrativas (Enterprise Tiering):**
   A Microsoft recomenda o isolamento estrito de ativos:
   * **Tier 0:** Domain Controllers, contas de Domain Admins e servidores PKI. Contas de Tier inferior (HelpDesk, estações comuns) jamais podem ter qualquer direito de escrita ou acesso sobre objetos de Tier 0.

### 🎯 Seu Objetivo

Mapear e analisar caminhos de escalação de privilégios em um ambiente de Active Directory modelado como grafo. Seu objetivo é investigar as relações de controle, direitos de escrita (GenericAll, WriteDacl) e sessões ativas para descobrir o caminho mais curto (*shortest path*) que permite a uma conta sem privilégios obter controle total sobre o domínio (*Domain Admin*).
