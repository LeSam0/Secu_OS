---
title: 'TP1 : Linux basiiiecs'

---

# TP1 : Linux basiiiecs

# Part I : Filtrey des paqueys

## 1. Intro
 
## 2. Conf

🌞 **Proposer une configuration restrictive de firewalld**

```bash
[toto@vbox ~]$ sudo firewall-cmd --zone=drop --list-all
drop (active)
  target: DROP
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 10.1.1.0/24
  services:
  ports: 22/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

# Part II : PAM

🌞 **Proposer une configuration de politique de mot de passe**

[Fichier configuration mot de passe](./Ressource/pwquality.conf)

# Part III : OpenSSH

## 1. Intro

## 2. Conf

🌞 **Proposer une configuration du serveur OpenSSH**

[Fichier configuration OpenSSH](./Ressource/sshd_config)

🌞 **Une fois en place, tu m'appelles en cours pour que je me connecte**

ip : 10.33.70.219 
port : 2200

# Part IV : Gestion d'utilisateurs

## 1. Gestion d'utilisateurs

🌞 **Gestion d'utilisateurs**

## 2. Gestion de permissions

🌞 **Gestion de permissions**

### 3. Sudo sudo sudo

🌞 **Gestion de `sudo`**

🌞 **Misconf ?**

🌞 **Proposer une meilleure conf**

# V. FTP

🌞 **Mettre en place un serveur FTP + TLS**

```bash
[toto@vbox ~]$ sudo dnf install vsftpd

[toto@vbox ~]$ sudo systemctl enable vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.

[toto@vbox ~]$ sudo firewall-cmd --zone=drop  --permanent --add-port=20/tcp
success

[toto@vbox ~]$ sudo firewall-cmd --reload
success

[toto@vbox ~]$ sudo mkdir /etc/ssl/private

[toto@vbox ~]$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt

[toto@vbox ~]$ sudo vi /etc/vsftpd/vsftpd.conf

[toto@vbox ~]$ sudo systemctl restart vsftpd

```

[Fichier configuration FTP](./Ressource/vsftpd.conf)