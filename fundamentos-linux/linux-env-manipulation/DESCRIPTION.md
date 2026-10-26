# Engenharia de Variáveis de Ambiente e Contexto de Execução

## 1. Briefing da Missão
Operador, durante a auditoria de um serviço de monitoramento interno da Cyberguard, identificou-se que o módulo de validação de integridade requer a presença de uma variável de ambiente operacional contendo o token de autorização da sessão de treinamento.

Sua missão tática é:
1. Compreender o funcionamento do bloco de ambiente dos processos no Linux (`environ`).
2. Configurar a variável `CYBERGUARD_TOKEN` com o valor `treinamento_2026`.
3. Executar o binário `/challenge/linux-env-manipulation` garantindo a herança do parâmetro pelo processo filho para capturar a flag.

---

## 2. Teoria Aprofundada

### 2.1 A Estrutura de Ambiente em Processos POSIX
No padrão POSIX, todo processo em execução possui um ponteiro global para uma matriz de strings terminadas em nulo (`char **environ`). Cada entrada possui o formato canônico `CHAVE=VALOR`:

```
Processo Shell (PID 1024)
├── Tabela de Símbolos Locais (não exportados)
└── Bloco de Ambiente (environ):
    ├── PATH=/usr/local/bin:/usr/bin:/bin
    ├── USER=cyberguard
    ├── HOME=/home/cyberguard
    └── CYBERGUARD_TOKEN=treinamento_2026   <--- [Herança via fork/execve]
```

### 2.2 Ciclo de Vida: Variáveis Locais vs `export`
Uma das fontes mais comuns de frustração em novos operadores é a distinção entre variáveis da shell e variáveis de ambiente:

| Método | Sintaxe | Visível na Shell Atual? | Herdado por Processos Filhos? |
| :--- | :--- | :---: | :---: |
| **Definição Local** | `TOKEN="segredo"` | **Sim** | **Não** |
| **Exportação Global** | `export TOKEN="segredo"` | **Sim** | **Sim** |
| **Injeção Efêmera** | `TOKEN="segredo" ./binario` | Não (apenas no filho) | **Sim (apenas no binário)** |

### 2.3 Relação com o Diagnóstico da Cyberguard
No diagnóstico do ciclo 2026.2, identificou-se que integrantes iniciantes confundiam a atribuição local simples (`VAR=123`) com a diretiva `export`, resultando em scripts de automação que falhavam silenciosamente porque os comandos subsequentes não recebiam as variáveis de configuração esperadas.

---

## 3. Passo a Passo de Resolução

1. **Tentativa inicial sem variável configurada:**
   Execute o binário para observar a validação do ambiente:
   ```bash
   /challenge/linux-env-manipulation
   ```
   *Resultado:* O sistema exibirá a aula didática alertando sobre a ausência da variável `CYBERGUARD_TOKEN`.

2. **Opção Recomendada (Injeção em Linha Única):**
   ```bash
   CYBERGUARD_TOKEN="treinamento_2026" /challenge/linux-env-manipulation
   ```
   *Vantagem:* A variável é instanciada apenas durante o ciclo de vida daquele processo específico, mantendo a shell limpa e sem poluição de ambiente.

3. **Opção Alternativa (Via `export` persistente na sessão):**
   ```bash
   export CYBERGUARD_TOKEN="treinamento_2026"
   env | grep CYBERGUARD
   /challenge/linux-env-manipulation
   ```

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição Operacional |
| :--- | :--- |
| `env` ou `printenv` | Lista todas as variáveis de ambiente ativas na sessão. |
| `export VAR=valor` | Torna a variável visível para todos os processos gerados a partir da shell atual. |
| `unset VAR` | Remove a variável do ambiente da sessão. |
| `cat /proc/$$/environ \| tr '\0' '\n'` | Inspeciona o bloco raw de ambiente do processo atual (`$$` = PID da shell). |

---

## 5. Aplicação Prática no Mundo Real

* **Cloud Security e Containers:** Em arquiteturas Kubernetes e Docker, segredos e tokens de banco de dados são injetados diretamente via variáveis de ambiente (`ENV`). Se a aplicação sofrer um ataque de execução remota de código (RCE) ou Server-Side Request Forgery (SSRF), o invasor poderá extrair todo o conteúdo de `/proc/self/environ`.
* **Privilege Escalation:** No Linux, administradores desatentos que configuram regras `sudo` com a diretiva `env_keep` ou `sudo -E` permitem que usuários comuns preservem variáveis como `LD_PRELOAD`, abrindo portas para escalação imediata para root.
