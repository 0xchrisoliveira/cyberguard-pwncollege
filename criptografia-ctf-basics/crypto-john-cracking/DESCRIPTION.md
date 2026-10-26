# Quebra de Hashes com John the Ripper / Hashcat

### 📖 Introdução Teórica
A quebra de hashes offline (*Offline Password Cracking*) é uma das disciplinas mais críticas de auditoria de credenciais e testes de intrusão corporativos.

**A Vantagem Operacional do Cracking Offline:**
Diferente de um ataque de força bruta online (que dispara requisições contra formulários web ou portas SSH e é rapidamente bloqueado por políticas de lockout, *fail2ban* ou alertas de SIEM), o ataque offline é executado **estritamente na máquina local do atacante**:
* O atacante obtém o hash (via dump de banco de dados, arquivo `/etc/shadow` do Linux, arquivo SAM/NTDS.dit do Windows ou ticket de serviço Kerberos).
* O ataque não transmite nenhum pacote pela rede corporativa e não pode ser interrompido por mecanismos de defesa de borda.
* O atacante pode testar bilhões de combinações por segundo aproveitando o paralelismo de suas GPUs e CPUs.

**Modalidades de Ataque Offline:**
1. **Ataque de Dicionário (Wordlist Attack):** Compara o hash alvo contra o hash de milhões de senhas reais compiladas de vazamentos históricos. A lista mais famosa da história da segurança é a **`rockyou.txt`**, que contém mais de 14 milhões de senhas exclusivas extraídas de um vazamento em 2009.
2. **Ataque com Regras de Mutação (Rule-Based Attack):** Aplica transformações dinâmicas sobre cada palavra da wordlist (como capitalizar a primeira letra, trocar letras por números estilo leetspeak `a -> @`, `e -> 3`, e anexar anos correntes como `2026!` ou `123`).
3. **Ataque por Máscara / Força Bruta (Mask / Brute-Force):** Testa exaustivamente todas as combinações de um padrão definido (ex: 8 dígitos numéricos ou 1 maiúscula + 5 minúsculas + 2 dígitos).

**As Duas Ferramentas Canônicas:**
* **John the Ripper (JtR):** Ferramenta clássica e extremamente versátil. Possui excelente suporte a múltiplos formatos em CPUs, suíte ampla de extratores de hash (`ssh2john`, `zip2john`, `keepass2john`) e detecção automática de formatos.
* **Hashcat:** O mecanismo de quebra acelerado por GPU mais poderoso do ecossistema, otimizado para drivers OpenCL e CUDA, capaz de utilizar o poder massivo de processamento vetorial de placas gráficas.

### 💻 Exemplos Práticos e Sintaxe
1. **Quebra de Hash MD5 com John the Ripper:**
   ```bash
   # 1. Salvar o hash alvo em um arquivo de texto:
   echo "e10adc3949ba59abbe56e057f20f883e" > /tmp/alvo.hash

   # 2. Executar o John the Ripper indicando o formato raw-md5:
   john --format=raw-md5 /tmp/alvo.hash

   # 3. Visualizar a senha quebrada a partir do banco interno (.potfile):
   john --show --format=raw-md5 /tmp/alvo.hash
   ```
   *Saída esperada:*
   ```text
   ?:123456

   1 password hash cracked, 0 left
   ```

2. **Utilizando John com a wordlist rockyou e regras de mutação:**
   ```bash
   john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt --rules /tmp/alvo.hash
   ```

3. **Quebra Equivalente com Hashcat (Aceleração por GPU):**
   ```bash
   # O modo -m 0 especifica o algoritmo MD5 puro; -a 0 especifica ataque de dicionário:
   hashcat -m 0 -a 0 /tmp/alvo.hash /usr/share/wordlists/rockyou.txt
   ```
   *Tabela de modos comuns do Hashcat:*
   * `-m 0`: MD5
   * `-m 100`: SHA-1
   * `-m 1400`: SHA-256
   * `-m 1000`: NTLM (Windows)
   * `-m 13100`: Kerberos 5 TGS-REP (Kerberoasting)
   * `-m 3200`: Bcrypt

### 🎯 Seu Objetivo

Realizar um ataque de dicionário offline contra um arquivo de hashes de senhas. Seu objetivo é utilizar a ferramenta John the Ripper em conjunto com uma lista de senhas conhecidas para quebrar o hash de uma credencial fraca, descobrindo a senha em texto claro e utilizando-a para autenticar no desafio e liberar a flag.
