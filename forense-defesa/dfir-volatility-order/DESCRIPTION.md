# Triage Emergencial: Respeitando a RFC 3227

### 📖 Introdução Teórica

Em computação forense e resposta a incidentes de segurança (DFIR - *Digital Forensics and Incident Response*), o primeiro contato com um sistema comprometido dita o sucesso ou o fracasso de toda a investigação pericial. Diante de um servidor sob ataque ativo, a reação instintiva de operadores destreinados costuma ser desligar imediatamente o equipamento da tomada ou iniciar a cópia de arquivos de log do disco rígido. Essa atitude catastrófica destrói as evidências mais ricas e definitivas do ataque.

Para padronizar internacionalmente a metodologia pericial, o IETF publicou a norma **RFC 3227** (*Guidelines for Evidence Collection and Archiving*), que institui o princípio universal da **Ordem de Volatilidade**:
> **"Colete sempre as evidências partindo da fonte mais volátil para a menos volátil."**

A volatilidade representa a velocidade com que uma evidência se degrada, é sobrescrita ou desaparece permanentemente caso o sistema perca alimentação elétrica, sofra reinicialização ou continue operando rotineiramente.

```
[ MAIS VOLÁTIL - Perece em microssegundos / requer energia ]
        ▲
        │  1. Registradores da CPU e Memória Cache
        │  2. Tabelas de Roteamento, Cache ARP, Conexões TCP/UDP Ativas, Tabela de Processos
        │  3. Memória Principal Física (RAM)
        │  4. Sistemas de Arquivos Temporários (/tmp, /dev/shm, swap, pagefile)
        │  5. Discos Não Voláteis (HDDs, SSDs, sistemas de arquivos, logs em /var/log)
        │  6. Configuração e Topologia Física da Rede
        │  7. Logs em Servidores Remotos (SIEM, syslog externo) e Tráfego Gravado
        │  8. Mídias Físicas de Arquivamento e Fitas de Backup a Frio
        ▼
[ MENOS VOLÁTIL - Persiste por meses ou anos sem alimentação elétrica ]
```

#### Por que a Memória RAM tem Prioridade Absoluta sobre o Disco?
A memória dinâmica de acesso aleatório (DRAM) depende de capacitores microscópicos que precisam ser atualizados milhares de vezes por segundo (*refresh cycles*). No instante em que um servidor é desligado ou reiniciado, o fornecimento elétrico cessa e o conteúdo da RAM dissipa-se em frações de segundo.

Em contrapartida, os discos magnéticos e memórias flash (SSDs) são dispositivos de armazenamento **não voláteis**: os bits gravados em clusters e blocos persistem intactos mesmo sem energia elétrica.

| Vetor de Análise | Memória RAM Física (Volátil) | Disco Rígido / Logs (Não Volátil) |
| :--- | :--- | :--- |
| **Persistência** | Destruída instantaneamente se o host for desligado. | Permanece gravada com o host desligado. |
| **Fileless Malware** | Executa estritamente no espaço de endereçamento (Process Hollowing, injeção de DLL reflexiva). | Não deixa nenhum executável gravado no sistema de arquivos. |
| **Credenciais de Acesso** | Tokens Kerberos, senhas em texto puro, chaves de sessão e hashes residem em memória. | Senhas raramente são registradas em arquivos de log limpos. |
| **Chaves de Criptografia** | Chaves simétricas ativas (ex: AES-256 de ransomware ou sessões TLS) vivem em buffers de processo. | Se a máquina for reiniciada, a chave de decifração é perdida para sempre. |
| **Conexões de Rede (C2)** | Descritores de sockets de túneis reversos e endereços IP de C2 residem na memória do kernel. | Podem não constar em logs locais se o invasor os desativou ou limpou. |

#### Relevância Ofensiva e Defensiva
* **Defensiva (Blue Team / DFIR):** O perito forense deve capturar a imagem da RAM (*dump*) antes de qualquer interação que possa alterar o estado do sistema, garantindo a integridade da **Cadeia de Custódia** através do cálculo de hash criptográfico imediato (SHA-256).
* **Ofensiva (Red Team):** Invasores avançados utilizam técnicas de malware em memória (*fileless*) justamente para evitar a detecção por antivírus tradicionais baseados em assinaturas de disco e garantir que suas pegadas desapareçam com um simples reboot.

---

### 💻 Exemplos Práticos e Sintaxe

A preservação forense da memória volátil envolve ferramentas dedicadas e procedimentos padronizados de integridade:

#### 1. Coleta Forense de RAM no Linux (LiME - Linux Memory Extractor)
O módulo de kernel LiME permite extrair a memória física de sistemas Linux com impacto mínimo no espaço de endereçamento:
```bash
# Inserir o módulo LiME indicando o destino do dump e o formato bruto (raw):
sudo insmod lime-$(uname -r).ko "path=/tmp/ram_forense.lime format=raw"

# Verificar se o despejo foi concluído com sucesso:
ls -lh /tmp/ram_forense.lime
```

#### 2. Coleta Forense de RAM no Windows
Utilizam-se executáveis portáteis que não exigem instalação prévia para não sobregravar clusters de memória:
```cmd
:: Extração física via WinPmem:
C:\Forensics> winpmem.exe -o C:\Forensics\mem_dump.raw

:: Extração via DumpIt:
C:\Forensics> DumpIt.exe /OUTPUT C:\Forensics\memory.dmp
```

#### 3. Garantia da Cadeia de Custódia (Cálculo de Hash Imediato)
Imediatamente após a geração da imagem forense, calcula-se o hash criptográfico para certificar judicialmente que a imagem não sofreu adulteração:
```bash
# Calcular e armazenar o hash SHA-256 da imagem da memória:
sha256sum /tmp/ram_forense.lime > /tmp/ram_forense.lime.sha256

# Inspecionar o hash gerado:
cat /tmp/ram_forense.lime.sha256
```

#### 4. Inspeção Volátil de Conexões de Rede (Antes de Interromper o Sistema)
```bash
# Listar conexões ativas com processos e sockets associados:
ss -tulpan
```

---

### 🎯 Seu Objetivo

Aplicar as diretrizes metodológicas da norma RFC 3227 sobre preservação de evidências digitais. Seu objetivo é organizar as diferentes fontes de evidência do sistema (registradores, memória RAM, tráfego de rede, armazenamento em disco e backups) em ordem estrita de volatilidade, garantindo que os dados mais efêmeros sejam coletados primeiro para validar a perícia e receber a flag.
