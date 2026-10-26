# Análise Estática Fundamental (Strings & Objdump)

### 📖 Introdução Teórica

A Engenharia Reversa de software é a disciplina dedicada a desconstruir um artefato compilado para compreender seus algoritmos internos, desvendar mecanismos de proteção ou identificar vulnerabilidades sem ter acesso prévio ao código-fonte original. No fluxo de análise de binários, o primeiro e mais seguro passo metodológico é a **Análise Estática**.

#### O que é Análise Estática e por que ela é Segura?
A Análise Estática consiste em examinar as estruturas internas, metadados, seções e instruções de máquina de um executável **sem colocá-lo em execução no processador**.
* **Segurança Absoluta:** Ao receber um artefato suspeito (como um ransomware, trojan bancário ou wiper), executá-lo diretamente pode causar a destruição irreversível do laboratório. A análise estática trata o binário puramente como uma sequência passiva de bytes em disco, eliminando qualquer risco de infecção acidental.
* **Visibilidade Integral:** Ferramentas de desmonte estático permitem visualizar 100% dos blocos de código e rotinas do programa — incluindo funções de depuração, backdoors e branches de erro que jamais seriam acionadas durante uma execução convencional.

```
+-----------------------------------------------------------------------------------------+
|                                ESTRUTURA DO FORMATO ELF                                 |
|                         (Executable and Linkable Format - Linux)                        |
+-----------------------------------------------------------------------------------------+
| CABEÇALHO ELF: Magic Bytes 7F 45 4C 46 ("\x7fELF"), Arquitetura (x86_64), Entry Point   |
+-------------+---------------------------------------------------------------------------+
| SEÇÃO .text | Instruções de máquina executáveis (código assembly gerado pelo compilador)|
+-------------+---------------------------------------------------------------------------+
| SEÇÃO       | Read-Only Data (Dados Somente Leitura):                                   |
| .rodata     | - Strings literais de texto ("Digite sua senha: ")                        |
|             | - Chaves criptográficas e tokens hardcoded ("REVERSE_STATIC_KEY_...")     |
+-------------+---------------------------------------------------------------------------+
| SEÇÃO .data | Variáveis globais e estáticas com valores iniciais definidos              |
+-------------+---------------------------------------------------------------------------+
| SEÇÃO .bss  | Variáveis não inicializadas (zeradas em memória no momento do carregamento|
+-----------------------------------------------------------------------------------------+
```

#### A Seção `.rodata` e o Erro das Chaves Hardcoded
Em programas escritos em C/C++, qualquer texto delimitado por aspas duplas (como `"senha_secreta"`) é posicionado pelo compilador diretamente na seção `.rodata` (*Read-Only Data*). Quando desenvolvedores negligentes incorporam segredos estáticos diretamente no código (*hardcoding*), esses artefatos ficam gravados em texto claro no arquivo executável, permitindo que qualquer analista os extraia em segundos com ferramentas elementares de linha de comando.

---

### 💻 Exemplos Práticos e Sintaxe

A caixa de ferramentas fundamental da análise estática no Linux é composta por utilitários da suíte GNU Binutils:

#### 1. Identificação do Tipo de Binário (`file`)
O utilitário `file` examina os magic bytes no cabeçalho do arquivo para identificar sua arquitetura de CPU, se é de 32 ou 64 bits, e se os símbolos de depuração foram removidos (*stripped*):
```bash
file /tmp/malware_sample.elf
```
*Saída típica:*
```text
/tmp/malware_sample.elf: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
```

#### 2. Extração de Strings Imprimíveis (`strings`)
O comando `strings` varre o arquivo em busca de sequências de 4 ou mais caracteres ASCII imprimíveis consecutivos terminados por byte nulo:
```bash
# Extrair todas as strings de todas as seções e filtrar por padrão:
strings -a /tmp/malware_sample.elf | grep REVERSE

# Exibir as strings acompanhadas de seu deslocamento (offset) em hexadecimal:
strings -t x /tmp/malware_sample.elf
```

#### 3. Desmonte em Assembly de Sintaxe Intel (`objdump`)
Para inspecionar o código de máquina desmontado em sintaxe legível Intel (em vez da notação AT&T com `%` e `$`), utilizamos o `objdump`:
```bash
# Desmontar todas as funções executáveis em sintaxe Intel:
objdump -d -M intel /tmp/malware_sample.elf

# Inspecionar especificamente o conteúdo da seção .rodata em hexadecimal e ASCII:
objdump -s -j .rodata /tmp/malware_sample.elf
```

#### 4. Mapeamento das Seções do Arquivo (`readelf`)
```bash
# Listar todas as seções ELF, endereços de memória virtual e permissões (W/A/X):
readelf -S /tmp/malware_sample.elf
```

---

### 🎯 O que fazer para resolver o desafio (Passo a Passo)

Siga os passos abaixo para conduzir a análise estática e extrair a chave do binário:

1. **Inicie o desafio para disponibilizar a amostra:**
   Execute `/challenge/rev-static-strings-objdump`:
   ```bash
   /challenge/rev-static-strings-objdump
   ```
   *O ambiente gerará o binário simulado para análise em `/tmp/malware_sample.elf`.*

2. **Inspecione a arquitetura do artefato com `file`:**
   Confirme que se trata de um binário ELF de 64 bits:
   ```bash
   file /tmp/malware_sample.elf
   ```

3. **Extraia as cadeias de texto da seção de dados com `strings`:**
   Execute o utilitário `strings` filtrando palavras-chave de interesse:
   ```bash
   strings -a /tmp/malware_sample.elf | grep REVERSE
   ```
   *Você também pode inspecionar diretamente o dump da seção `.rodata`:*
   ```bash
   objdump -s -j .rodata /tmp/malware_sample.elf
   ```

4. **Identifique a chave estática:**
   O terminal revelará a constante gravada na seção de dados:
   ```text
   REVERSE_STATIC_KEY_984
   ```

5. **Submeta a chave ao validador da missão:**
   Envie a chave encontrada como parâmetro de execução do desafio:
   ```bash
   /challenge/rev-static-strings-objdump REVERSE_STATIC_KEY_984
   ```
   *Ou insira-a no prompt interativo se invocado sem argumentos.*

6. **Capture e registre a flag:**
   O binário homologará o resultado da análise estática e exibirá a flag:
   ```text
   [+] DEBRIEFING TÉCNICO: CHAVE ESTÁTICA LOCALIZADA COM SUCESSO!
   [FLAG DE VALIDAÇÃO CYBERGUARD]:
   pwn.college{...}
   ```
   Copie a flag no formato `pwn.college{...}` e insira no sistema de pontuação.
