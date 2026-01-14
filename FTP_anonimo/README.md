# FTP-ANONIMO
First part of FTP practice

# STEPS

1. Infrastructure creation with Vagrant
2. Ansible files creation
3. Deployment and error checkings
4. VM access and file checking
5. Testings from the host

## 1. Infrastructure creation with Vagrant

The first step will be the Vagrantfile creation. We must install a Debian, in our case we chose "bookworm64" and set the hostname "mirror.sistema.sol".
Configure the network and add the provision with Ansible.

(enlace al vagrantfile)

## 2. Ansible files creation

Now, we must create all the Ansible files required to do the practice. I tried to do it in a modular way to keep everything organised.

Create th inventory and the main playbook.
In taht playbook we will define the roles. In this case we only have one: vsftpd.

In the folder roles --> vsftpd, we have the following sub-directories each one with its own files.

In the tasks directory we will place the playbook that contains all the task that are needed. The tasks will be executed in the following order:

- vsftpd installation
- FTP directory creation
- copy the welcome message which will be on the folder "files"
- run the configuration file for vsftpd and wait for the handler to notify (that handler will be in the folder handlers)
- check that vsftpd service is active

The configuration file for vsftpd will be located on the folder templates and will be done in Jinja in order for it to be dynamic and reusable.
The values are set in defaults/main.yml