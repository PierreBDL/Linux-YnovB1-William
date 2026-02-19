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
# Mis à jour des paquets
sudo apt update -y
sudo apt upgrade -y

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

- Logs ssh

``` bash
# On regarde les logs ssh
sudo tail -f /var/log/auth.log
```

![](/TP4-170226/Images/Partie5/log-ssh.png)

On peut voir les différentes connexions et tentatives de connexion à la VM.

- fail2ban

``` bash
# On installe fail2ban
sudo apt install fail2ban -y

# On vérifie que fail2ban est actif
sudo systemctl status fail2ban

# On regarde le tableau de bord de fail2ban
sudo fail2ban-client status sshd
```

On test pour se faire bannir :
ssh pierr@10.151.168.93 -p 2222

![](/TP4-170226/Images/Partie5/ban.png)

Je me suis bien fait bannir

## Partie 6 – Tunnel SSH

1. 

``` bash
# On créait un tunnel local pour accèder au site d'apache
ssh -L 8080:localhost:80 vm

# Mis à jour des paquets
sudo apt update -y
sudo apt upgrade -y

# On installe apache juste pour tester le localhost
sudo apt install apache2

# On autorise le traffic apache pour éviter que le pare-feu ne bloque le traffic
sudo ufw allow 'Apache Full'

# On vérifie
sudo ufw status

# On vérifie qu'on écoute bien au port 80
sudo ss -tlnp | grep apache2
```

![](/TP4-170226/Images/Partie6/apache-full.png)
![](/TP4-170226/Images/Partie6/verif-traffic.png)

Résultat :

![](/TP4-170226/Images/Partie6/site-apache.png)

2. 

``` bash
# On créer le tunel distant pour le port 22 (ssh)
ssh -R 9090:localhost:22 vm
```

## Partie 7 – Nginx et HTTPS

``` bash
# Mis à jour des paquets
sudo apt update -y
sudo apt upgrade -y

# Installation de Nginx
sudo apt install nginx -y

# On désactive apache qui occupe les ports que va utiliser nginx
sudo systemctl stop apache2
sudo systemctl disable apache2

# On active nginx
sudo systemctl start nginx

# On regarde si nginx fonctionne
sudo systemctl status nginx
``` 

Configuration et mise en place du site :

``` bash
# On créer un dossier qui va accueillir le site web de test
sudo mkdir -p /var/www/site-tp

# On créer un fichier html pour le site
echo "<h1>Bienvenue sur le site TP Nginx</h1>" | sudo tee /var/www/site-tp/index.html

# On configure nginx
echo "server {
    listen 80;
    server_name localhost;
    root /var/www/site-tp;
    index index.html;
}" | sudo tee /etc/nginx/sites-available/site-tp

# On créait le lien symbolique (raccourci)
sudo ln -s /etc/nginx/sites-available/site-tp /etc/nginx/sites-enabled/

# On vérifie qu'il n'y ait pas de problème de syntaxe
sudo nginx -t

# On redémarre nginx pour qu'il prenne en compte les modifiactions effecutées
sudo systemctl restart nginx
```

Certificat SSL :

``` bash
# Création du dossier où il y aura les données des certifiacats SSL
sudo mkdir -p /etc/nginx/ssl

# Déplacement dans le dossier
cd /etc/nginx/ssl

# Génération du certifiat ssl de 1 an
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout site-tp.key -out site-tp.crt

# On configure nginx
echo 'server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /etc/nginx/ssl/site-tp.crt;
    ssl_certificate_key /etc/nginx/ssl/site-tp.key;

    root /var/www/site-tp;
    index index.html;
}

server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
}' | sudo tee /etc/nginx/sites-available/site-tp

# On vérifie s'il y a des fautes
sudo nginx -t

# On redémarre le service
sudo systemctl restart nginx
``` 

- Sur windows :
``` bash
# On vérifie si on peut se connecter sur le site de la VM
curl -k https://10.151.168.93
```

Résultat :

![](/TP4-170226/Images/Partie7/resultat-ssl.png)
![](/TP4-170226/Images/Partie7/resultat-ssl-web.png)

On peut voir que le contenu est le bon.


## Partie 8 – Firewall et permissions

``` bash
# On autorise le traffic nginx pour éviter que le pare-feu ne bloque le traffic
sudo ufw allow 'Nginx Full'

# On vérifie
sudo ufw status

# On donne la propriété du dossier à Nginx
sudo chown -R www-data:www-data /var/www/site-tp

# Donne les droits de lecture et d'exécution à tout le monde
chmod -R 755 /var/www/site-tp
```

Résultat :

![](/TP4-170226/Images/Partie8/status-ufw.png)




