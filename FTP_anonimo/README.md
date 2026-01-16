# FTP-ANONIMO
This is the first part of the practice where we must create an anonymous FTP server and check that it works correctly.

## STEPS

1. Infrastructure creation with Vagrant
2. Ansible files creation
3. Deployment and error checkings
4. Testings from the host

### 1. Infrastructure creation with Vagrant

The aim is to create a modular infrastructure using **Vagrant** and **Ansible** in order to make the project re-usable (it will come handy for the second part).

The first step will be the Vagrantfile creation. We must install a Debian, in our case we chose ***bookworm64*** and set the hostname ***mirror.sistema.sol***.
Configure the network and add the provision with Ansible.

[Vagrantfile](Vagrantfile)

### 2. Ansible files creation

Now, we must create all the Ansible files required to do the practice. I tried to do it in a modular way to keep everything organised following this  structure:

```text

ansible/
├── inventory.yml
├── playbook.yml
└── roles/
    └── vsftpd/
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── templates/
        │   └── vsftpd.conf.j2
        ├── files/
        │   └── message.txt
        └── defaults/
            └── main.yml

```

First, we created the inventory and the main playbook.
In that playbook we will define the roles. In this case we only have one: ***vsftpd***.

[Main playbook](ansible/playbook.yml)
[Inventory](ansible/inventory.yml)

In the folder /roles/vsftpd, we have the following sub-directories each one with its own files.

In the **tasks** directory we will place the playbook that contains all the task that are needed. The tasks will be executed in the following order:

- vsftpd installation
- FTP directory creation
- copy the welcome message which will be on the folder "files"
- run the configuration file for vsftpd and wait for the handler to notify (that handler will be in the folder handlers)
- check that vsftpd service is active

[Playbook tasks](ansible/roles/vsftpd/tasks/main.yml)

The configuration file for vsftpd will be located on the folder **templates** and will be done in Jinja in order for it to be dynamic and reusable. The values are set in **defaults** folder.

[Configuration file](ansible/roles/vsftpd/templates/vsftpd.conf.j2)
[Current configuration values](ansible/roles/vsftpd/defaults/main.yml)

We also have a folder called **files** where we will have the message that will appear whe connecting to the server and the **handlers** folder where the handlers will be located.

[Message file](ansible/roles/vsftpd/files/message.txt)
[Handlers file](ansible/roles/vsftpd/handlers/main.yml)

### 3. Deployment and error checkings

Once everything is OK, run the Vagrantfile using `vagrant up` and check for possible errors.
If there are no errors, enter the VM with `vagrant ssh` and check the status of vsftpd with the command `systemctl status vsftpd` and its configuration file `cat /etc/vsftpd.conf`. This should be running correctly.

<details>
<summary><code>systemctl status vsftpd"</code></summary>

> **Verification:** Checks the status  of vsftpd
>
**Output:**
```bash

● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-01-16 13:07:57 UTC; 4min 7s ago
    Process: 2221 ExecStartPre=/bin/mkdir -p /var/run/vsftpd/empty (code=exited, >
   Main PID: 2222 (vsftpd)
      Tasks: 1 (limit: 496)
     Memory: 1008.0K
        CPU: 4ms
     CGroup: /system.slice/vsftpd.service
             └─2222 /usr/sbin/vsftpd /etc/vsftpd.conf

```
</details>

<details>
<summary><code>ss -tlpn | grep :21</code></summary>

> **Verification:** Checks the status of the port of ftp
>
**Output:**
```text
LISTEN 0      32           0.0.0.0:21        0.0.0.0:*
```
</details>

<details>
<summary><code>cat /etc/vsftpd.conf</code></summary>

> **Verification:** Checks the configuration file of vsftpd
>
**Output:**
```text

# Configuracion general

# Solo IPv4
listen=YES
listen_ipv6=NO

# Mensaje de bienvenida
ftpd_banner=-- Mirror de Opensuse para 'sistema.sol'


# Usuarios

# Permitir solo usuarios anonimos
anonymous_enable=YES

# Denegar usuarios locales
local_enable=NO

# No permitir escritura
write_enable=NO

# Directorio raiz anonimo
anon_root=/srv/ftp

# Mensaje al entrar en directorios

dirmessage_enable=YES
message_file=.message


# Limites

# Maximo de clientes simultaneos
max_clients=200

# Ancho de banda anonimo: 50 KB/s
anon_max_rate=51200

# Timeout de inactividad (30 segundos)
idle_session_timeout=30


# Seguridad

anon_upload_enable=NO
anon_mkdir_write_enable=NO
anon_other_write_enable=NO

```
</details>

### 4. Testings from the host

- We must have installed ftp in our physical machine. Run `ftp --version to check it`.
- Run the command `ftp 192.168.56.101` to connect to the ftp server
-  Enter the name ***anonymous*** to enter as an anonymous user and for the password click enter.

<details>
<summary><code> ftp 192.168.56.10</code></summary>

> **Verification:** Connects to th ftp server.
>
**Expected Output:**
```text

alumnom@a112-pc05:~/Documentos/ftp/FTP_anonimo$ ftp 192.168.56.10
Connected to 192.168.56.10.
220 -- Mirror de Opensuse para 'sistema.sol'
Name (192.168.56.10:alumnom): anonymous
331 Please specify the password.
Password: 
230---
230--- Servidor anónimo
230--- Máximo de 200 conexiones activas simultáneas.
230--- Ancho de banda de 50KB por usuario
230--- Timeout de inactividad de 30 segundos
230---
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 

```
</details><br>

- Try to write creating any file with the `mkdir prueba` command and check that we cannot do it

<details>
<summary><code>mkdir test</code></summary>

> **Verification:** Try to create a directory
>
**Output:**
```text

ftp> mkdir
(directory-name) test
550 Permission denied.
ftp> 

```
</details><br>

- Stay inactive for 30 seconds and then try to do something to check that the timeout works.

<details>
<summary>Stay inactive for a while</summary>

> **Verification:** Checks if the inactivity time works.
>
**Output:**
```text

ftp> mkdir
(directory-name) prueba
421 Timeout.
ftp> 

```
</details>


