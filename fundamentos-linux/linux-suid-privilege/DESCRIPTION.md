# O Poder do SUID (Set User ID e Escalação de Privilégios)

## 1. Briefing da Missão
Operador, a infraestrutura do pwn.college protege a `/flag` definindo permissões restritas exclusivas ao superusuário `root` (`-r-------- 1 root root`). Para que desafios de treinamento permitam a leitura autorizada da flag, o sistema utiliza o recurso mais poderoso do modelo de segurança UNIX: **o bit SUID (Set User ID)**.

Sua missão tática é:
1. Compreender a mecânica do bit SUID e a distinção fundamental entre Real UID (RUID) e Effective UID (EUID).
2. Auditar permissões no sistema de arquivos localizando binários com bit SUID.
3. Executar o binário `/challenge/linux-suid-privilege`, inspecionar o seu EUID operacional e resgatar a flag sob autorização do kernel.

---

## 2. Teoria Aprofundada

### 2.1 O que é o Bit SUID?
Em circunstâncias normais, um processo herda o identificador do usuário que o invocou (RUID). Se você é o usuário `hacker` (UID 1000), o processo roda com UID 1000 e não pode ler arquivos restritos ao `root` (UID 0).

Quando um binário possui o bit **SUID (Set User ID)** configurado:
* O kernel do Linux substitui o **Effective UID (EUID)** pelo UID do **proprietário do arquivo** durante a chamada `execve()`.
* Se o dono do arquivo for o `root`, o programa executa com privilégios de superusuário, independentemente de quem o disparou!

```
[ Usuário: hacker (UID 1000) ]
              │
              │ Dispara: /usr/bin/passwd (dono: root, bit SUID ativo)
              ▼
    ┌────────────────────────────────────────────────────────┐
    │ Processo em Execução pelo Kernel:                      │
    │   • Real UID (RUID)      = 1000 (hacker)               │
    │   • Effective UID (EUID) = 0    (root) <--- DECISIVO!  │
    └────────────────────────────────────────────────────────┘
              │
              │ O Kernel permite a escrita restrita:
              ▼
   [ Arquivo /etc/shadow ] (Acesso Concedido ao EUID 0)
```

### 2.2 Tabela dos Bits Especiais de Execução

| Bit Especial | Notação Octal | Máscara Simbólica | Efeito Prático | Risco Ofensivo |
| :---: | :---: | :---: | :--- | :--- |
| **SUID** | `4xxx` | `-rwsr-xr-x` | Executa com o UID do dono (geralmente root) | Escalação Local de Privilégio (LPE) |
| **SGID** | `2xxx` | `-rwxr-sr-x` | Executa com o GID do grupo do arquivo | Herança forçada de grupo |
| **Sticky Bit**| `1xxx` | `drwxrwxrwt` | Apenas o dono pode apagar seus arquivos na pasta | Segurança em `/tmp` |

### 2.3 Relação com o Diagnóstico da Cyberguard
No relatório diagnóstico, constatou-se que embora integrantes reconhecessem a vulnerabilidade de funções em C, havia uma desconexão total com a forma como o sistema operacional concede acesso efetivo a arquivos protegidos. Compreender SUID é a chave mestra para entender toda a trilha de exploração binária (pwn) e pós-exploração no Linux.

---

## 3. Passo a Passo de Resolução

1. **Auditoria do binário do desafio:**
   Inspecione os atributos do binário:
   ```bash
   ls -la /challenge/linux-suid-privilege
   ```
   *Note a presença do bit `s` ou o wrapper de execução com permissões privilegiadas.*

2. **Varredura global de binários SUID no sistema:**
   Descubra quais outros programas no sistema operacional possuem o bit SUID:
   ```bash
   find / -perm -4000 -type f 2>/dev/null
   ```
   *Você identificará utilitários como `/usr/bin/passwd`, `/usr/bin/sudo` e o wrapper `/usr/bin/exec-suid`.*

3. **Execução do binário do desafio:**
   ```bash
   /challenge/linux-suid-privilege
   ```
   *O programa exibirá o RUID e o EUID em tempo real, validará a transição de privilégios e imprimirá a flag.*

---

## 4. Dicas e Comandos de Terminal

| Comando | Descrição |
| :--- | :--- |
| `id` | Imprime o UID, GID e grupos suplementares do usuário atual. |
| `chmod u+s <arquivo>` | Adiciona o bit SUID ao arquivo (equivalente a `chmod 4755`). |
| `chmod u-s <arquivo>` | Remove o bit SUID do arquivo. |
| `find / -perm -u=s -type f 2>/dev/null` | Sintaxe simbólica alternativa para buscar binários SUID. |

---

## 5. Aplicação Prática no Mundo Real

* **Escalação Local de Privilégios (GTFOBins):** No mundo real de testes de invasão e auditorias internas, se um administrador conceder bit SUID indevidamente a utilitários comuns (como `find`, `nmap`, `python`, `cp`), atacantes podem abusar desses binários para ler arquivos confidenciais ou criar shells interativas de root sem precisar de senha.
* **Defesa e Hardening (CIS Benchmark):** Ambientes corporativos seguros implementam auditorias periódicas via scripts de SIEM para alertar sobre qualquer novo binário SUID introduzido no sistema (`auditd`).
