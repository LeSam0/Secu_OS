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

```sh 
ssh-keygen -t rsa -b 2048
ssh-copy-id toto@192.168.56.10
```

2.

Voici les ports et services ouvert sur ma machine

```sh 

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

sudo firewall-cmd --add-port=50/tpc
sudo firewall-cmd --add-port=50/udp

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

3. SELinux peut fonctionner dans trois modes: désactivé, permissif ou appliqué. Le basculement entre les modes peut nécessiter un réamorçage.
"Désactivé" signifie qu'aucune vérification d'accès ou consignation n'est effectuée.
"Permissive" signifie que les violations d'accès sont consignées, mais qu'elles sont autorisées à se produire.
"Mise en application" signifie que la règle est appliquée et que l'accès sera refusé s'il n'a pas été autorisé dans la règle.

( source: [IBM](https://www.ibm.com/docs/fr/db2/11.5.0?topic=security-enhanced-linux-selinux) )

4. 
