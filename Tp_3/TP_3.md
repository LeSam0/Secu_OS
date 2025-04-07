# TP 3 Hardening des systèmes avec SElinux

## 3. Consignes

### 3.1 Installation du systèmes d'exploitations : 

1. Installation du rocky 9.5 en version minimal sans interface graphique. 

2. J'ai effectuer c'est deux commande afin de mettre a jour la vm
```sh 
    sudo dnf update
    sudo dnf upgrade
```

### 3.2 Sécurisation de l'administration du serveur : 

1. Voici la configuration sshd 

je me suis aussi créer un clé RSA, RSA n'était plus le meilleur niveau sécurite de nos jour, j'aurais pu prendre autre chose mais j'ai juste suivi la doc ANSSI. 

[Fichier de config ssh](./Ressource/sshd_config.txt)

```sh 
ssh-keygen -t rsa -b 2048
ssh-copy-id toto@192.168.56.10
```

2. Voici les ports et services ouvert sur ma machine

```sh 
sudo firewall-cmd --list-all
public (active)
    target: default
    icmp-block-inversion: no
    interfaces: enp0s3 enp0s8
    sources:
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

je n'ai laisser que le port ssh car je n'ai besoin que de celui la


### 3.3 Installation d'un serveur Web

1. Installation simple, j'avais juste a ouvrir le port a la fin

```sh 
sudo dnf install httpd

sudo firewall-cmd --add-port=80/tpc
sudo firewall-cmd --add-port=80/udp

sudo firewall-cmd --reload
```

Voici le resultat

```sh 
curl http://192.168.56.10
<!doctype html>
<html>
[...]
</body>
</html>
```

2. SELinux etait déjà installer, la commande ```sestatus``` permet de la vérifier

3.  SELinux peut être exécuté dans l'un des trois modes suivants : disabled, permissive ou enforcing.

L'utilisation du mode disabled signifie qu'aucune règle de la stratégie SELinux n'est appliquée et que votre système n'est pas protégé. Nous vous déconseillons donc d'utiliser le mode disabled.

Avec le mode permissive, SELinux est actif, la stratégie de sécurité est chargée, le système de fichiers est étiqueté et des entrées sont consignées en cas de refus d'accès. Toutefois, la stratégie n'est pas appliquée et aucun accès n'est donc réellement refusé.

En mode enforced, la stratégie de sécurité est appliquée. Chaque accès qui n'est pas explicitement autorisé par la stratégie est refusé. 

( source: [SUSE](https://documentation.suse.com/fr-fr/sle-micro/6.0/html/Micro-selinux/index.html) )

4. SELinux bloquera les accès non autorisé ce qui peux empêcher des programmes de fonctionner correctement


### 3.4 Modification d'un profil SELinux

1. le context des fichiers de config de httpd est le suivant 
```sh 
ls -lZ /etc/httpd/conf/
total 28
-rw-r--r--. 1 root root system_u:object_r:httpd_config_t:s0 12005 Jan 22 01:22 httpd.conf
-rw-r--r--. 1 root root system_u:object_r:httpd_config_t:s0 13430 Jan 22 01:24 magic

```

pour les fichiers html, c'est le suivant 

```sh
ls -lZ /var/www/html/
total 4
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 63 Apr  3 14:21 index.html
```

2. Le context du service apache est le suivant 

```sh
ps axZ | grep httpd
system_u:system_r:httpd_t:s0       2013 ?        Ss     0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0       2019 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0       2020 ?        Sl     0:02 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0       2021 ?        Sl     0:02 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0       2022 ?        Sl     0:02 /usr/sbin/httpd -DFOREGROUND
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 2360 pts/0 S+   0:00 grep --color=auto httpd
```

3. J'active le mode "enforce" en le changant dans le fichier de conf 
```sh
vi /etc/selinux/config
SELINUX=enforcing
```

Le serveur web fonctionne toujours

4. Je deactive le mode "enforce" en le changant dans le fichier de conf 
```sh
vi /etc/selinux/config
SELINUX=disabled
```
Je créer les dossiers deplace mon fichier html et modifie ma conf httpd

```sh
mkdir /srv/srv
mkdir /srv/srv/srv_1
mv /var/www/html/index.html /srv/srv/srv_1/index.html
```

Une fois le serveur redemarer, je peux le contacter et j'ai bien une reponse

```sh
curl http://192.168.56.10
<html><body><div>HELLO WORLD !! IT'S ToTo</div></body></html>
```

5. Quand je réactive le mode "enforce", je peut toujours contacter le serveur, mais je tombe sur la page par defaut de apache. Comme le serveur n'a pas le droit pour accèder au dossier que je viens de créer ou ce trouve mon fichier html, le serveur prend son fichier par défaut.

```sh 
curl http://192.168.56.10
<!doctype html>
<html>
[...]
</body>
</html>
```

6. Sealert permet de comprendre plus facilement les problèmes lier a SELinux en analysant le log SELinux et en nous donnant les commandes permetant de corriger

On annalyse d'abord les logs 
```sh
sudo sealert -a /var/log/audit/audit.log
100% done
found 2 alerts in /var/log/audit/audit.log
--------------------------------------------------------------------------------

SELinux is preventing /usr/sbin/httpd from getattr access on the file /srv/srv/srv_1/index.html.

*****  Plugin catchall_labels (83.8 confidence) suggests   *******************

If you want to allow httpd to have getattr access on the index.html file
Then you need to change the label on /srv/srv/srv_1/index.html
Do
# semanage fcontext -a -t FILE_TYPE '/srv/srv/srv_1/index.html'

```

On change l'etat pour le passer de unconfined a httpd_sys_content_t.
C'est deux commandes permet de corriger le problème en changeant le type et en appliquant les nouveaux contextes

```sh
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/srv/srv_1(/.*)?"

sudo restorecon -Rv /srv/srv/srv_1/
Relabeled /srv/srv/srv_1 from system_u:object_r:var_t:s0 to system_u:object_r:httpd_sys_content_t:s0
Relabeled /srv/srv/srv_1/index.html from unconfined_u:object_r:var_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0

```

7. Une fois le correctif fait, on peut reaccèder au serveur et a notre ficher html

```sh
curl http://192.168.56.10
<html><body><div>HELLO WORLD !! IT'S ToTo</div></body></html>
```

### 3.5 Durcissement de la configuration de SELinux

Les bonnes pratiques sont disponibles sur le site 
[Bonnes pratiques CIS](https://cissecurity.org/benchmark/rocky_linux)

On verifie que SELinux est activer et en mode "enforcing", ce qu'on a configurer précedament.

On verifie qu'aucun service esr en "unconfined_service", ce qu'on a corriger précedament
