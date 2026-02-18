# TP – Administration SSH et Serveur Web Nginx

## Objectifs

- Installer et configurer une VM Ubuntu.  
- Mettre en place un serveur SSH sécurisé avec authentification par clé.  
- Transférer des fichiers et analyser les logs.  
- Installer Nginx et configurer HTTPS avec certificat auto-signé.  
- Configurer le firewall et sécuriser les permissions.  

---

## Partie 1 – Mise en place de l’environnement virtualisé

1. Installez **VirtualBox** et créez une VM Ubuntu :  
   - 2 Go RAM minimum, 20 Go disque.  
   - Réseau : Bridged Adapter.  

2. Vérifiez que la VM a une IP accessible depuis la machine hôte.  
   **Pistes** : `ip a`, `ping <IP_VM>`  
   **Documentation** : [VirtualBox User Manual](https://www.virtualbox.org/manual/UserManual.html)  

---

## Partie 2 – Serveur SSH

1. Installez le serveur SSH sur la VM.  
   **Indice** : chercher le paquet `openssh-server`.  

2. Vérifiez que le service SSH fonctionne et écoute sur un port.  
   **Piste** : `systemctl status` + `ss` ou `netstat`.  

3. Connectez-vous depuis la machine hôte :  
```bash
ssh etudiant@<IP_VM>
```
4. Générez une clé SSH sur la machine cliente et copiez-la sur le serveur pour tester la connexion sans mot de passe.
Pistes : `ssh-keygen`, `ssh-copy-id`

## Partie 3 – Sécurisation SSH

Modifiez la configuration SSH sur le serveur pour renforcer la sécurité :

1. **Interdisez l’accès root**.  
2. **Désactivez l’authentification par mot de passe**.  
3. **Changez le port par défaut** (22) pour réduire les tentatives de brute-force.  
4. **Testez la connexion** avec le nouveau port depuis la machine cliente.  
5. **Créez un alias SSH** dans `~/.ssh/config` pour simplifier les connexions.

**Piste** : recherchez le fichier `sshd_config` et les options `PermitRootLogin`, `PasswordAuthentication`.

## Partie 4 – Transfert de fichiers

Transférez un fichier et un dossier depuis la machine cliente vers le serveur :

- **SCP** : `scp fichier.txt serveur-tp:/home/etudiant/`  
- **SFTP** : explorez les commandes `put`, `get`, `ls` pour transférer et naviguer sur le serveur.  
- **RSYNC** : synchronisez un dossier entre client et serveur.

**Pistes** : utilisez les options `-a` (archive), `-v` (verbose), `-z` (compression) pour RSYNC.

## Partie 5 – Analyse des logs et sécurité

- Suivez les logs d’authentification pour observer les connexions SSH :  
```bash
sudo tail -f /var/log/auth.log
```
- Installez Fail2Ban et testez un bannissement après plusieurs tentatives échouées.

## Partie 6 – Tunnel SSH

- Créez un **tunnel local** pour accéder à un service web distant depuis la machine cliente.

- Créez un **tunnel distant** pour permettre l’accès SSH au client via le serveur.

## Partie 7 – Nginx et HTTPS

- Installez **Nginx** sur la VM.

- Créez un site test dans `/var/www/site-tp` et un fichier `index.html` avec un message de bienvenue.

- Configurez Nginx pour servir ce site sur **HTTP**.

- Générez un **certificat auto-signé** pour HTTPS et configurez la **redirection HTTP → HTTPS**.  
Piste :  
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048
```
- Testez le site depuis le client :
```
curl -k https://<IP_VM>
```
## Partie 8 – Firewall et permissions

- Autorisez **Nginx** dans le firewall (ports HTTP/HTTPS).  
Piste :  
```bash
sudo ufw allow 'Nginx Full'
```
- Vérifiez les permissions sur /var/www/site-tp pour que Nginx puisse lire les fichiers.

Pistes : utiliser chown et chmod pour définir le propriétaire et les droits.

## Partie 9 – Validation finale

- **SSH** fonctionnel sur port personnalisé et authentification par clé uniquement.  
- **Fail2Ban** actif et opérationnel.  
- **Transferts de fichiers** fonctionnels (SCP, SFTP, RSYNC).  
- **Nginx** accessible en HTTP et HTTPS avec redirection automatique HTTP → HTTPS.  
- **Certificat SSL** auto-signé valide.  
- **Firewall** configuré et **permissions** correctes sur `/var/www/site-tp`.