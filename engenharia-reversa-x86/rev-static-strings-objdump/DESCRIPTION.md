# Análise Estática Fundamental (Strings & Objdump)

## Briefing da Missão
A divisão de Inteligência Cibernética da Cyberguard interceptou um artefato malicioso suspeito de operar como dropline e coletor de credenciais corporativas. O arquivo foi isolado em `/tmp/malware_sample.elf`.

Por questões de segurança operacional e risco biológico digital, o protocolo exige que o artefato **NÃO seja executado** inicialmente no ambiente de produção. Sua missão como especialista em engenharia reversa é aplicar métodos de **Análise Estática**, examinando as seções do arquivo ELF com ferramentas de linha de comando (`file`, `strings`, `readelf`, `objdump`), encontrar a chave de autenticação estática que o desenvolvedor deixou hardcoded e submetê-la ao validador da missão.

---

## Teoria Aprofundada & Relação com o Diagnóstico 2026.2

No diagnóstico do ciclo 2026.2, **87,5% da equipe cometeu erros na distinção entre análise estática e dinâmica**, um dos conceitos basilares de engenharia reversa e análise de ameaças.

### Análise Estática vs. Análise Dinâmica

| Critério | Análise Estática | Análise Dinâmica |
| :--- | :--- | :--- |
| **Execução do Código** | **Nunca executa o binário.** O arquivo é lido como um fluxo passivo de bytes. | **Executa o binário** em ambiente controlado (GDB, sandbox, VM com snapshot). |
| **Segurança** | 100% segura contra detonações acidentais e infecções. | Requer isolamento rígido de rede e contenção perimetral. |
| **Visibilidade** | Cobre todos os blocos de código e caminhos lógicos da aplicação. | Apenas observa o caminho percorrido naquela execução específica. |
| **Desafios** | Dificuldade contra binários ofuscados, packers (UPX) ou código polimórfico. | Evasões de anti-debugging (`ptrace`, detecção de breakpoints). |
| **Ferramentas Típicas** | `strings`, `file`, `readelf`, `objdump`, Ghidra, IDA Pro. | GDB, GEF, `ltrace`, `strace`, Frida, x64dbg. |

### Estrutura de Seções do Formato ELF (Executable and Linkable Format)

```
+-------------------------------------------------------------+
|                      CABEÇALHO ELF                          |
|             (Magic Bytes: 7F 45 4C 46 = "\x7fELF")          |
+-------------------------------------------------------------+
|  .text      | Instruções de máquina da CPU (Código C/Asm)   |
+-------------+-----------------------------------------------+
|  .rodata    | Read-Only Data (Strings literais, constantes, |
|             | chaves hardcoded como "REVERSE_STATIC_KEY_...")|
+-------------+-----------------------------------------------+
|  .data      | Variáveis globais inicializadas pelo dev      |
+-------------+-----------------------------------------------+
|  .bss       | Memória não inicializada (zerada pelo loader) |
+-------------------------------------------------------------+
```

---

## Passo a Passo de Resolução

1. Execute o desafio para gerar o artefato e verificar os parâmetros:
   ```bash
   /challenge/rev-static-strings-objdump
   ```
2. Inspecione o tipo do arquivo isolado:
   ```bash
   file /tmp/malware_sample.elf
   ```
3. Extraia todas as sequências de texto imprimíveis da seção de dados com o utilitário `strings`:
   ```bash
   strings -a /tmp/malware_sample.elf | grep REVERSE
   ```
4. Se desejar inspecionar o código de máquina desmontado em sintaxe Intel:
   ```bash
   objdump -d -M intel /tmp/malware_sample.elf
   ```
5. Observe a chave identificada:
   ```text
   REVERSE_STATIC_KEY_984
   ```
6. Envie a chave ao binário do desafio:
   ```bash
   /challenge/rev-static-strings-objdump REVERSE_STATIC_KEY_984
   ```
7. O debriefing técnico será exibido e a flag de validação será impressa.

---

## Dicas e Comandos de Terminal

- **Localizando o Offset Hexadecimal de Strings:**
  Use `strings -t x` para saber o endereço exato do byte dentro do arquivo:
  ```bash
  strings -t x /tmp/malware_sample.elf
  ```
- **Lendo os Cabeçalhos de Seções com Readelf:**
  ```bash
  readelf -S /tmp/malware_sample.elf
  ```
- **Filtrando Diretamente com Grep Binário:**
  ```bash
  grep -ao "REVERSE_STATIC_[0-9A-Z_]*" /tmp/malware_sample.elf
  ```

---

## Aplicação Prática no Mundo Real

Em auditorias de segurança de aplicações móveis e IoT (Internet of Things), bem como na resposta a incidentes, analistas frequentemente encontram chaves de API, senhas de banco de dados e credenciais de nuvem (AWS Access Keys) gravadas diretamente no binário pelo desenvolvedor. A análise estática com `strings` e `objdump` é a primeira técnica adotada em qualquer rotina de triagem de segurança.
