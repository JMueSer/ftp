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

## 🔍 Technical Verification Audit

Click on any command below to reveal the expected terminal output and verification logic.

<details>
<summary><code>vagrant ssh -c "sudo openssl x509 -in /etc/ssl/certs/example.test.pem -text -noout | grep Subject"</code></summary>

> **Verification:** Checks if the SSL certificate contains the required geographical and organizational data.
>
**Expected Output:**
```text
Subject: C = ES, ST = Granada, L = Granada, O = Sistema, CN = ftp.example.test