# Engenharia de Variáveis de Ambiente e Contexto de Execução

### 📖 Introdução Teórica

No sistema operacional Linux, cada processo possui um bloco de memória reservado para armazenar seu contexto operacional na forma de pares chave-valor denominado **Bloco de Variáveis de Ambiente** (`char **environ`). Quando um processo pai (como a sua shell Bash) cria um processo filho através das chamadas de sistema `fork()` e `execve()`, o ambiente é herdado pelo processo recém-criado.

```
Processo Pai: Bash (PID 1024)
├── Variáveis Locais de Shell (não exportadas, restritas à shell)
└── Bloco de Ambiente Global (environ):
    ├── PATH=/usr/local/bin:/usr/bin:/bin
    ├── USER=hacker
    ├── HOME=/home/hacker
    └── CYBERGUARD_TOKEN=treinamento_2026  ──[ Herança via execve() ]──> Processo Filho (PID 1025)
```

#### Variáveis Locais vs. `export` vs. Injeção Inline
Uma das falhas mais frequentes em administração e segurança de sistemas é presumir que qualquer variável declarada no terminal será visível aos programas executados a seguir:

| Método | Sintaxe no Terminal | Visível na Shell Atual? | Herdada por Processos Filhos? |
| :--- | :--- | :---: | :---: |
| **Atribuição Local** | `VAR="teste"` | **Sim** | **Não** (invisível para binários externos) |
| **Exportação Global** | `export VAR="teste"` | **Sim** | **Sim** (herdada por todos os filhos futuros) |
| **Injeção Efêmera** | `VAR="teste" ./binario` | **Não** (apenas no binário) | **Sim** (isolada estritamente no subprocesso) |

#### Relevância Ofensiva e Defensiva
* **Segurança Ofensiva (Red Team / PrivEsc):** 
  - **PATH Hijacking:** Se o `PATH` contiver diretórios graváveis pelo usuário antes dos diretórios do sistema (ex: `PATH=.:$PATH`), binários privilegiados podem ser induzidos a executar binários maliciosos locais em vez dos comandos do sistema.
  - **Injeção via Variáveis Dinâmicas:** Variáveis como `LD_PRELOAD` ou `LD_LIBRARY_PATH` forçam o carregador dinâmico a carregar bibliotecas arbitrárias antes de qualquer outra. Em regras `sudo` mal configuradas que preservam o ambiente (`env_keep`), isso concede escalação imediata para root.
* **Segurança Defensiva (Blue Team / Cloud Security):** Em contêineres Docker e orquestrações Kubernetes, segredos de bancos de dados, chaves de API e tokens JWT são costumeiramente injetados via variáveis de ambiente. Qualquer vulnerabilidade de LFI (Local File Inclusion) ou RCE que permita inspecionar o arquivo `/proc/self/environ` pode expor integralmente as credenciais corporativas.

---

### 💻 Exemplos Práticos e Sintaxe

O gerenciamento de variáveis de ambiente no Linux envolve os seguintes comandos e técnicas:

#### Comandos e Sintaxe Operacional
* `printenv` ou `env`: Lista todas as variáveis de ambiente exportadas na sessão atual.
* `printenv <VAR>`: Exibe o valor de uma variável específica.
* `export <VAR>="<valor>"`: Promove a variável para a tabela global de ambiente, garantindo que processos filhos a herdem.
* `unset <VAR>`: Remove a variável do ambiente da sessão.
* `<VAR>="<valor>" <comando>`: Executa o comando injetando a variável exclusivamente naquele ciclo de vida, sem poluir a shell atual.

#### Snippets de Terminal e Saídas Observadas

1. **Demonstração de Falha com Variável Apenas Local:**
   ```bash
   $ MINHA_VAR="cyberguard"
   $ python3 -c "import os; print(os.environ.get('MINHA_VAR'))"
   None
   ```
   *Como a variável não foi exportada, o subprocesso Python não a encontrou.*

2. **Demonstração com `export` (Herança Confirmada):**
   ```bash
   $ export MINHA_VAR="cyberguard"
   $ python3 -c "import os; print(os.environ.get('MINHA_VAR'))"
   cyberguard
   ```

3. **Injeção Inline sem Poluição de Escopo:**
   ```bash
   $ TOKEN="efemero" python3 -c "import os; print(os.environ.get('TOKEN'))"
   efemero
   $ echo $TOKEN
   (vazio)
   ```

4. **Inspeção Forense via Pseudo-Filesystem `/proc`:**
   ```bash
   $ cat /proc/$$/environ | tr '\0' '\n' | grep PATH
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
   ```
   *O pseudofilesystem armazena variáveis terminadas pelo byte nulo (`\0`); o comando `tr` as substitui por quebras de linha.*

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Siga estas instruções para suprir a variável esperada e resgatar a flag:

1. **Inspecione a exigência do binário:**
   Execute `/challenge/linux-env-manipulation` sem alterar o ambiente:
   ```bash
   /challenge/linux-env-manipulation
   ```
   *O programa indicará que a variável `CYBERGUARD_TOKEN` está ausente ou com valor incorreto.*

2. **Método 1: Resolução Elegante com Injeção Inline (Recomendado):**
   Defina a variável imediatamente antes do comando em uma única linha:
   ```bash
   CYBERGUARD_TOKEN="treinamento_2026" /challenge/linux-env-manipulation
   ```

3. **Método 2: Resolução com `export` na Sessão:**
   Exporte a variável para a shell e execute o binário:
   ```bash
   export CYBERGUARD_TOKEN="treinamento_2026"
   /challenge/linux-env-manipulation
   ```
   *(Opcional) Verifique se a variável está ativa antes da execução:*
   ```bash
   printenv CYBERGUARD_TOKEN
   ```

4. **Capture e submeta a flag:**
   O binário validará a variável no dicionário de ambiente e liberará a flag:
   ```text
   [+] Variavel de ambiente CYBERGUARD_TOKEN identificada!
   pwn.college{...}
   ```
   Copie a sequência no padrão `pwn.college{...}` e insira no portal de treinamento.
