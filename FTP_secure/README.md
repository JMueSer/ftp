# Secure FTP Server Deployment Project

## 📖 Project Overview
The goal of this project is to deploy a fully functional and secure FTP server using **vsftpd** on a **Debian** virtual machine. The entire infrastructure is automated using **Vagrant** for virtualization and **Ansible** for configuration management, ensuring a reproducible and hardened environment.

### 📝 Project Requirements
The practice required the implementation of the following specific features:
1.  **Messaging & Banners**: 
    - A custom welcome banner for every connection.
    - Directory-specific information messages triggered upon entering folders.
2.  **Performance Control**: 
    - Limiting the server to a maximum of 15 simultaneous clients.
    - Bandwidth throttling to prioritize resources: 5MB/s for local users and 2MB/s for anonymous users.
3.  **User Isolation (Selective Chroot)**: 
    - Implementation of a "jail" (chroot) system where users are restricted to their home directories.
    - Configuration of a specific exception list to allow the user **Maria** to navigate the entire system root, while others like **Luis** remain jailed.
4.  **Cryptographic Security**: 
    - Forced SSL/TLS encryption for both authentication and data transfer.
    - Automated generation of a self-signed certificate with specific geolocation and organizational data (Granada, Sistema).

---

## 🔍 Technical Audit & Expected Results

Follow these commands to verify the server's compliance with the requirements.

### 1. SSL/TLS Certificate Audit
Verify that the automated "bot" generated the certificate with the correct identity:
<details>
<summary><code>vagrant ssh -c "sudo openssl x509 -in /etc/ssl/certs/example.test.pem -text -noout | grep Subject"</code></summary>

> **Verification:** Checks if the SSL certificate contains the required geographical and organizational data.
>
**Expected Output:**
```text
Subject: C = ES, ST = Granada, L = Granada, O = Sistema, CN = ftp.example.test
```
</details>

### 2. Network Service & Port Audit
Check if the vsftpd service is correctly running and listening for connections:
<details>
<summary><code>vagrant ssh -c "sudo ss -tlnp | grep :21"</code></summary>

> **Verification** Confirms the FTP service is active and listening on the standard port 21.
>
**Expected Output:**
```text
LISTEN 0 32 *:21 *:* users:(("vsftpd",pid=654,fd=3))
```
</details>

### 3. Banners and Welcome Messages
Test the custom connection banner and the directory-specific message:
<details>
<summary><code>ftp 192.168.56.10</code> (Connect as anonymous)</summary>

> **Verification** Tests the custom Welcome Banner and the Directory Message.
>
**Expected Output:**
```text
Connected to 192.168.56.10.
220 -- Welcome to the FTP server of 'sistema.sol'
Name (192.168.56.10:user): anonymous
331 Please specify the password.
230-Bienvenido al directorio público de sistema.sol. Disfruta de tu estancia.
230 Login successful.
```
</details>

### 4. User Isolation (Luis - Jailed)
Verify that standard users are restricted to their HOME directory:
<details>
<summary><code>lftp -u luis,password123 -e "set ssl:verify-certificate no" 192.168.56.10</code></summary>

> **Verification** Confirms Luis is correctly jailed in his HOME using SSL.
>
**Expected Output:**
```text
lftp luis@192.168.56.10:~> ls
-rw-r--r--    1 1001     1001           26 Jan 14 12:05 luis1.txt
lftp luis@192.168.56.10:/> cd /etc
cd: Access failed: 550 Failed to change directory. (User is jailed)
```
</details>

### 5. Chroot Exception (Maria - Unjailed)
Verify that the exception list allows Maria to access the system root:
<details>
<summary><code>lftp -u maria,password123 -e "set ssl:verify-certificate no" 192.168.56.10</code></summary>

> **Verification** Confirms Maria is an exception and can escape the jail.
>
**Expected Output:**
```text
lftp maria@192.168.56.10:~> cd /
lftp maria@192.168.56.10:/> ls
drwxr-xr-x   20 0        0            4096 Jan 14 12:05 bin
drwxr-xr-x    78 0        0            4096 Jan 14 12:05 etc
drwxr-xr-x    3 0        0            4096 Jan 14 12:05 home
# (Full system root is accessible)
```
</details>

### 6. Security & Key Permissions
Verify that sensitive files (SSL keys) are correctly protected:
<details>
<summary><code>vagrant ssh -c "sudo ls -l /etc/ssl/private/example.test.key"</code></summary>

> **Verification** Ensures the private key has restricted permissions.
>
**Expected Output:**
```text
-rw------- 1 root root 1704 Jan 14 12:05 /etc/ssl/private/example.test.key
```
</details>