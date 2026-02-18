## **TP 2 - Vendredi 23 Janvier 2026** <br>
**Fais sur une VM Fedora** <br>
**Pierre BDL**

---

## Partie 1 – Mise en place de l’environnement virtualisé

1. 
![](/TP4-170226/Images/Partie1/virtualBox.png)

2. 
On vérifie l'adresse IP attribuée à la VM
![](/TP4-170226/Images/Partie1/ipa.png)

On ping la VM depuis l'hôte
![](/TP4-170226/Images/Partie1/ping.png)

## Partie 2 – Serveur SSH

1. 
``` bash
# Installation du service ssh
sudo apt install openssh-server -y
```

2. 
``` bash
# On regarde si le service ssh est actif
sudo systemctl status ssh.service
```

3. On se connecte en ssh à la VM depuis windows
``` bash
ssh pierre@10.157.3.93
```

4. 

- Sur linux :

``` bash
# On configure les fichiers de ssh
sudo nano /etc/ssh/sshd_config
```

![](/TP4-170226/Images/Partie2/config.png)

``` bash
# On redémarre ssh
sudo systemctl restart ssh.service
```

- Sur windows :

```bash
# On génère les clés ssh
ssh-keygen

# On copie la clé dans la VM linux
ssh-copy-id pierre@10.157.3.93

# On retente de se connecter
ssh pierre@10.157.3.93
```

![](/TP4-170226/Images/Partie2/ssh-sans-cle.png)

## Partie 3 – Sécurisation SSH

1. 

``` bash
# On configure les fichiers de ssh
sudo nano /etc/ssh/sshd_config

# On décommente ou ajoute la ligne :
PermitRootLogin no

# On sauvegarde et on redémarre ssh
sudo systemctl restart ssh.service
```

![](/TP4-170226/Images/Partie3/connect-root.png)


2. 

``` bash
# On configure les fichiers de ssh
sudo nano /etc/ssh/sshd_config

# On décommente ou ajoute la ligne :
PasswordAuthentication no

# On sauvegarde et on redémarre ssh
sudo systemctl restart ssh.service
```

3. 

``` bash
# On configure les fichiers de ssh
sudo nano /etc/ssh/sshd_config

# On décommente ou ajoute la ligne :
Port 22
Port 2222

# On désactive ssh.socket
sudo systemctl disable ssh.socket

# On sauvegarde et on redémarre ssh
sudo systemctl restart ssh.service

# On vérifie qu'on écoute bien les ports 2222 et 22
sudo ss -tulpn | grep ssh
```

4. 

![](/TP4-170226/Images/Partie3/connect-2222.png)

5.

- Sur windows :

``` bash
# On configure l'alias dans C:\Users\Nom\.ssh\config :
Host vm
    HostName 10.157.3.93
    User pierre
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# On tente de se connecter
ssh vm
```

## Partie 4 – Transfert de fichiers

- SCP

``` bash
# Transfer du fichier avec SCP
scp '.\Desktop\YNOV\Sauvegarder infos git.txt' vm:/home/pierre
```

![](/TP4-170226/Images/Partie4/scp.png)

- SFTP

``` bash
# Connexion à la VM
sftp vm

# Lister les fichiers de la VM
ls

# Télécharger le fichier depuis le serveur (exemple fichier test)
get test

# Importer un fichier dans le serveur
put '.\Desktop\YNOV\Sauvegarder infos git.txt'
```

![](/TP4-170226/Images/Partie4/sftp.png)

- RSYNC

``` bash
# Syncroniser les fichiers
rsync -avz ./Desktop/YNOV/rsyncTest.txt vm:/Desktop/
```

## Partie 5 – Analyse des logs et sécurité

``` bash
# On regarde les logs ssh
sudo tail -f /var/log/auth.log
```

![](/TP4-170226/Images/Partie5/log-ssh.png)

On peut voir les différentes connexions et tentatives de connexion à la VM.

