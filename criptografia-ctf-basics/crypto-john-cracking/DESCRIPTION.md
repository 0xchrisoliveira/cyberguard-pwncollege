# Quebra de Hashes com John the Ripper / Hashcat (Crypto John Cracking)

## Briefing da Missão
* **Codinome:** Operação Crack The Citadel
* **Classificação:** Confidencial / Treinamento de Red Team & Auditoria de Senhas
* **Alvo:** Hash Criptográfico MD5 Extraído de Servidor Legado
* **Objetivo:** Utilizar ferramentas especializadas de quebra offline de credenciais (*John the Ripper* ou *Hashcat*) com listas de palavras (*wordlists*) para recuperar a senha em texto claro que originou o hash `e10adc3949ba59abbe56e057f20f883e`.

Em auditorias de segurança e pentests internos, equipes de Red Team frequentemente encontram dumps de senhas ou arquivos de hashes expostos. A capacidade de operar ferramentas de quebra offline de alta performance com dicionários e regras de mutação é uma habilidade fundamental.

---

## Teoria Aprofundada

### 1. Mecânica dos Ataques de Dicionário Offline
Diferente de ataques de força bruta online (que disparam tentativas contra um formulário de login HTTP e são bloqueados por limites de taxa ou *fail2ban*), a **quebra offline de hashes ocorre inteiramente na máquina do atacante**:

```
[ DUMP DE BANCO DE DADOS ] ──> Extrai Hash: e10adc3949ba59abbe56e057f20f883e
                                                 │
                                                 ▼
[ WORDLIST (rockyou.txt) ] ──> Candidato 1: "admin"    ──> MD5: 21232f297a... != Alvo
                               Candidato 2: "password" ──> MD5: 5f4dcc3b5a... != Alvo
                               Candidato 3: "123456"   ──> MD5: e10adc3949... == ALVO ENCONTRADO!
```

O atacante não envia nenhum pacote pela rede corporativa durante a fase de cracking.

### 2. Comparativo: John the Ripper vs Hashcat

| Critério | John the Ripper (JtR) | Hashcat |
| :--- | :--- | :--- |
| **Foco de Execução** | Otimizado primariamente para **CPU** (com suporte OpenCL) | Otimizado agressivamente para **GPU** (CUDA / OpenCL) |
| **Identificação de Formatos**| Detecção automática inteligente de hashes | Exige especificação manual do modo numérico (`-m <modo>`) |
| **Uso Típico** | Arquivos Unix `/etc/shadow`, arquivos ZIP/RAR protegidos, hashes rápidos | Grandes volumes de senhas corporativas, WPA2, NTLM e Kerberos |
| **Regras de Mutação** | Excelente motor de regras (`--rules`) | Altamente customizável com regras combinatórias |

### 3. Tabela de Modos Populares no Hashcat

| Tipo de Hash | Modo Hashcat (`-m`) | Formato de Exemplo |
| :--- | :---: | :--- |
| **MD5 Puro** | `0` | `e10adc3949ba59abbe56e057f20f883e` |
| **SHA-1** | `100` | `2fd4e1c67a2d28fced849ee1bb76e7391b93eb12` |
| **SHA-256** | `1400` | `5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8` |
| **NTLM (Windows)** | `1000` | `cc32e442d8002b0e43384ec1da76743b` |
| **Kerberos 5 TGS (Kerberoast)**| `13100` | `$krb5tgs$23$*...` |
| **Bcrypt** | `3200` | `$2b$12$...` |

---

## Passo a Passo de Resolução

1. Execute o desafio para inspecionar o hash alvo:
   ```bash
   /challenge/crypto-john-cracking
   ```
2. Caso precise consultar a sintaxe do John the Ripper:
   ```bash
   /challenge/crypto-john-cracking --help
   ```
3. Para quebrar o hash no ambiente Linux com **John the Ripper**:
   ```bash
   # 1. Salvar o hash em um arquivo:
   echo "e10adc3949ba59abbe56e057f20f883e" > /tmp/hash.txt

   # 2. Executar o john indicando o formato raw-md5:
   john --format=raw-md5 /tmp/hash.txt

   # 3. Exibir o resultado obtido:
   john --show --format=raw-md5 /tmp/hash.txt
   ```
4. A senha revelada será:
   ```text
   123456
   ```
5. Insira a senha recuperada no prompt do desafio:
   ```text
   123456
   ```
6. Ou invoque o binário diretamente via CLI:
   ```bash
   /challenge/crypto-john-cracking 123456
   ```
7. O sistema confirmará a validação do hash e exibirá a flag.

---

## Dicas e Comandos Práticos

* **Ataque com Hashcat em GPU:**
  ```bash
  hashcat -m 0 -a 0 /tmp/hash.txt /usr/share/wordlists/rockyou.txt
  ```
* **Usando Regras de Mutação com John the Ripper:**
  Caso a senha do alvo seja uma variação com números ou símbolos (ex: `Senha2026!`), as regras transformam a wordlist em runtime:
  ```bash
  john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt --rules /tmp/hash.txt
  ```
* **Extraindo hashes de arquivos protegidos:**
  A suíte do John possui dezenas de utilitários `*2john`:
  * `zip2john protegido.zip > zip.hash`
  * `rar2john backup.rar > rar.hash`
  * `pdf2john documento.pdf > pdf.hash`
  * `ssh2john id_rsa > rsa.hash`

---

## Aplicação Prática no Mundo Real
Em auditorias de *Active Directory* e testes de intrusão interna, a recuperação de senhas através de quebra offline de hashes NTLM (extraídos via DCSync ou dumping de LSASS) e tickets Kerberos (Kerberoasting) é uma das etapas mais críticas da cadeia de ataque (Kill Chain). 

Para a defesa corporativa, a lição é direta: implementar políticas de senhas baseadas em comprimento mínimo (14+ caracteres) e frases secretas (*passphrases*), eliminando combinações previsíveis presentes em listas como a `rockyou.txt`.
