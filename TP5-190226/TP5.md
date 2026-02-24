## **TP 2 - Vendredi 20 Février 2026** <br>
**Fais sur une VM pfSense et 2 VMs Ubuntu** <br>
**Pierre BDL**

---

## Partie 1 – Prise en main et sécurisation

### 1. Accès à l’interface

![](/TP5-190226/Images/Partie1/interface-web.png)

- Questions :

1. Quelle est l’adresse IP du LAN  ?
> C'est 192.168.56.101/24 sur le réseau 192.168.56.100
![](/TP5-190226/Images/Partie1/ip.png)

2. Quelle est l’adresse IP du WAN ?
> C'est 10.0.2.15/24.
![](/TP5-190226/Images/Partie1/ip.png)

3. Pourquoi utilise-t-on HTTPS ?
> Pour sécuriser les données qui circulent entre la VM pfsense et le navigateur web sur l'hôte.

4. Pourquoi faut-il changer les identifiants par défaut sur un pare-feu ?
> Car les mots de passe par défaut ne sont pas sécurisés; en effet, si deux utilisateurs installent pfsense, les mots de passe seront les mêmes donc ça ouvre la voie à du piratage. De plus, ce serait ironique que le pare-feu, qui protège tout le réseau, se fasse pirater à cause d'un mot de passe non sécurisé.

### 2. Sécurisation de l’accès administrateur

![](/TP5-190226/Images/Partie1/interface-utilisateurs.png)
![](/TP5-190226/Images/Partie1/mdp-change.png)

- Questions :

1. Où se gèrent les utilisateurs ?
> Aller dans Système puis dans Gestionnaire d'utilisateurs.
![](/TP5-190226/Images/Partie1/interface-utilisateurs.png)

2. Qu’est-ce qu’un mot de passe robuste ?
> C'est un mot de passe long, avec une grande diversité de caractères (majuscules, minuscules, chiffres et caractères spéciaux). Il doit également être unique et spécial (éviter les "password", "1234567890", "azerty", etc).

3. Pourquoi sécuriser en priorité l’accès admin sur un équipement réseau ?
> Car l'admin a tout les droits sur le réseau. Donc, si quelqu'un y a accès, il a accès a tout le réseau et donc a tout les utilisateurs. C'est donc plus rentable, pour un pirate, de prendre le contrôle de tous les utilisateurs via un seul piratage plutot que plusieurs utilisateurs mais au prix de plusieurs piratages pas forcement utiles.

## Partie 2 – Comprendre les interfaces réseau

### 3. Vérification des interfaces

![](/TP5-190226/Images/Partie2/assignation-ip.png)

- Questions :

1. Quelle interface permet l’accès Internet ?
> L'interface WAN (Wide Area Network) donc l'interface avec l'adresse 10.0.2.15/24.

2. Quelle interface correspond au réseau interne ?
> L'interface LAN (Local Area Network) donc l'interface avec l'adresse 192.168.56.101/24.

3. Que se passerait-il si les interfaces étaient inversées ?
> Si on n'échange pas aussi les adresses, ce serait comme si le pare-feu fonctionnait à l'envers.

## Partie 3 – Configuration des services réseau

### 4. DHCP

On désactive le dhcp de virtual box :

![](/TP5-190226/Images/Partie3/dhcp-virtual-box.png)

On met en place la plage dans services>dhcp server :

![](/TP5-190226/Images/Partie3/dhcp-pfsense.png)

On fait comme au début : on change l'adresse de la vm pfsense mais on met une adresse statique (192.168.56.99/24) 

- Questions :

1. Pourquoi utiliser DHCP plutôt qu’une IP fixe ?
> Comme ça, on n'a pas besoin de configurer les adresses à chaque fois qu'il y a un nouvel appareil. Ça évite aussi les risques de conflit (deux appareils avec la même adresse).

2. Quelle plage d’adresses choisir ?
> Les adresses entre 192.168.56.100/24 et 192.168.56.200/24.

3. Quelles adresses faut-il éviter d’inclure dans la plage ?
> Les adresses telles que : 192.168.56.0 (réseau), 192.168.56.254, 192.168.56.255 (broadcast) et celle de pfsense (192.168.56.99).

- Vérification :

![](/TP5-190226/Images/Partie3/virtual-box.png)
![](/TP5-190226/Images/Partie3/ipa.png)

Ubuntu a l'adresse 192.168.56.102/24 qui se situe bien dans l'intervalle du serveur dhcp (entre 192.168.56.100 et 192.168.56.254).

### 5. DNS

On configure les dns à utiliser sur pfsense :
![](/TP5-190226/Images/Partie3/dns-addresses.png)

On configure une règle pour autoriser la communication entre le LAN et le WAN :
![](/TP5-190226/Images/Partie3/rule-dns.png)

On configure la route par défaut pour sortir du LAN sur la VM Ubuntu :
![](/TP5-190226/Images/Partie3/routes.png)

On peut ping le DNS de google :
![](/TP5-190226/Images/Partie3/ping-8888.png)


- Questions :

1. Pourquoi un pare-feu peut-il jouer le rôle de serveur DNS ?
> Car il regarde les adresses donc il peut, s'il a une base de données, regarder si l'adresse est sûr ou non.

2. Que se passe-t-il si le DNS ne fonctionne pas mais que le ping vers 8.8.8.8 fonctionne ?
> Ca veut dire que le DNS a du mal à traduire l'url en ip ou alors qu'il a du mal à renvoyer le résultat.

## Partie 4 – Autoriser l’accès Internet

### 6. Règles de pare-feu

On passe les protocoles accepté à TCP/UDP. TCP pour les DNS et UDP pour les sites :
![](/TP5-190226/Images/Partie4/dns-pfsense.png)

``` bash
# On met l'adresse de la VM pfsense en tant que dns dans le fichier /etc/resolv.conf du client
sudo nano /etc/resolv.conf

# On ajoute l'adresse de la vm pfsense
nameserver 192.168.56.99

# On ping google
ping google.fr
```

![](/TP5-190226/Images/Partie4/dns.png)

Résultat :
![](/TP5-190226/Images/Partie4/ping-google.png)


- Questions :

1. Quelle doit être la source ?
> La source doit être le LAN car les requêtes sont effectuées par les machines du LAN.

2. Quelle doit être la destination ?
> La destination doit être le WAN car les requêtes sont pour des ip externes au LAN. Sur la règle, ce doit être any car on ne sait pas à quoi ressemble le réseau de destination.

3. Faut-il autoriser tous les protocoles ?
> Non, pour le dns, on peut juste autoriser l'UDP. Accepter tous les protocoles peut nuire à la sécurité et augmenter le nombre de failles. Il vaut mieux donner le strict minimum.

- Test :

1. Ping vers pfSense :

![](/TP5-190226/Images/Partie4/ping-pfsense.png)


2. Ping vers 8.8.8.8 :

![](/TP5-190226/Images/Partie4/ping-8888.png)


3. Test DNS :

![](/TP5-190226/Images/Partie4/test-dns.png)


4. Accès web :

![](/TP5-190226/Images/Partie4/test-web.png)


### 7. NAT

Pour voir la configuration, il faut aller dans Firewall>NAT>Outbound

![](/TP5-190226/Images/Partie4/nat.png)


- Questions :

1. Pourquoi le NAT est-il nécessaire avec une interface WAN en NAT ?
> Déjà qu'on manque d'adresse IPV4, si on n'avait pas de NAT, le nombre d'appareils pouvant se connecter, avec une IPV4, à Internet serait de 4,3 milliards en comptant les PC, les téléphones, les serveurs, etc. Le NAT est donc nécessaire pour avoir une IPV4, car ça permet à la box, ici pfsense, d'avoir un réseau local avec les appareils de la maison et une adresse publique pour communiquer sur Internet. C'est donc nécessaire pour pouvoir avoir un accès à Internet.

2. Quelle est la différence entre NAT automatique et manuel ?
> En manuel, on peut interdire certains réseaux de se connecter à Internet tandis que l'automatique permet d'autoriser toutes les sorties.

3. Comment vérifier qu’une traduction d’adresse a lieu ?
> On peut aller sur pfsense dans diagnostics>states>states et regarder la traduction. Par exemple, sur la photo en dessous, on peut voir que l'adresse 192.168.56.102 a été traduite en 10.0.2.15.
![](/TP5-190226/Images/Partie4/nat-vm.png)


## Partie 5 – Filtrage

### 8. Blocage d’un site spécifique

On va bloquer youtube :

![](/TP5-190226/Images/Partie5/youtube-ban.png)

Résultat :

![](/TP5-190226/Images/Partie5/youtube.png)
![](/TP5-190226/Images/Partie5/youtube2.png)

- Questions :

1. Faut-il bloquer par IP ou par nom de domaine ?
> Par nom de domaine car le site peut être migré sur un autre serveur avec une nouvelle IP donc le blocage ne sera pas efficace. De plus, des gros sites, qui sont fréquentés par beaucoup de monde, sont hébergés sur plusieurs serveurs avec des IP différentes, mais avec un même nom de domaine donc bloquer l'IP ne servirait à rien. Enfin, on peut utiliser un VPN pour contourner le pare-feu et donc le blocage.

2. Que se passe-t-il si le site utilise HTTPS ?
> Lorsque l'utilisateur tente de se connecter au site en http, le site affiche "La connexion a été réinitialisée" tandis qu'en https, le site affiche "Impossible de se connecter".
![](/TP5-190226/Images/Partie5/question2.png)

3. Pourquoi le blocage par IP peut-il être contourné ?
> Le site peut être migré sur un autre serveur avec une nouvelle IP donc le blocage ne sera pas efficace. De plus, des gros sites, qui sont fréquentés par beaucoup de monde, sont hébergés sur plusieurs serveurs avec des IP différentes, mais avec un même nom de domaine donc bloquer l'IP ne servirait à rien. Enfin, on peut utiliser un VPN pour contourner le pare-feu et donc le blocage.


### 9. Blocage d’une catégorie de sites (jeux d’argent)

On va dans Firewall > Aliases et on créait un alias avec tout les sites qu'on veut bloquer.

![](/TP5-190226/Images/Partie5/alliases.png)

Puis on met une règle pour bloquer :

![](/TP5-190226/Images/Partie5/rule_jeux.png)

Résultat :

![](/TP5-190226/Images/Partie5/winamax-bloque.png)

- Questions :

1. Pourquoi ne pas créer une règle par site ?
> Sinon, on devrait avoir autant de règles que de site qu'on veut bloquer. Donc pour éviter le bazar et le fait que ce soit long et pénible à faire, on fait un alias où on peut juste rajouter le site. En plus, c'est trié par catégories (Exemple : "Jeux-Argent") donc c'est plus facile pour la gestion.

2. Où se créent les alias ?
> On va dans Firewall puis dans Aliases.

3. Comment vérifier qu’une règle bloque réellement le trafic ?
> On va sur une machine du réseau et on tente d'accèder à un site bloqué avec un ping :
![](/TP5-190226/Images/Partie5/winamax-bloque.png)


## Partie 6 – Aller plus loin (partie plus tendue)

### 10. Blocage par catégorie (réseaux sociaux)

On va dans Firewall>Aliases et on créait un alias avec tout les sites qu'on veut bloquer.

![](/TP5-190226/Images/Partie6/reseaux_sociaux-alias.png)

Puis on met une règle pour bloquer :

![](/TP5-190226/Images/Partie6/rule-reseaux.png)

Résultat :

![](/TP5-190226/Images/Partie6/insta-bloque.png)

Logs disponibles dans Status > System Logs > Firewall > Normal View :

![](/TP5-190226/Images/Partie6/insta-log.png)

- Question :

1. Que se passe-t-il si la règle est placée sous une règle "Pass Any" ?
> Si c'est le cas, "Pass any" est prioritaire et autorisera la requête.


### 11. Règles horaires

On créer l'orloge dans Firewall > Schedules > Edit

![](/TP5-190226/Images/Partie6/calendrier.png)

On ajoute le calendrier dans les paramètres avancés de la règle Réseaux Sociaux :

![](/TP5-190226/Images/Partie6/calendrier-rule.png)
![](/TP5-190226/Images/Partie6/calendrier-rule-general.png)

Puisqu'il est 16h, on peut se connecter à instagram :

![](/TP5-190226/Images/Partie6/calendrier-resultat.png)

- Question :

1. Pourquoi les règles horaires sont-elles utiles en entreprise ?
> Grâce à ça, les entreprises peuvent par exemple bloquer les réseaux sociaux pendant les heures de travail et les autoriser pendant les pauses. Ça permet donc aux entreprises de pousser les employés à être concentré dans leur travail.


### 12. Serveur web local

- Sur la VM Ubuntu 2

``` bash
# On définit la passerelle par défaut pour accèder à internet
sudo ip route add default via 192.168.56.99

# Je met pfsense en DNS
sudo nano /etc/resolv.conf

# On ajoute
nameserver 192.168.56.99

# Mise à jour des paquets
sudo apt update -y
sudo apt upgrade -y

# J'installe nginx pour héberger le site
sudo apt install nginx -y
```

- PFSense

Configuration du calendrier :

![](/TP5-190226/Images/Partie6/calendrier-site.png)

Création de la règle :

![](/TP5-190226/Images/Partie6/regle-site.png)
![](/TP5-190226/Images/Partie6/calendrier-rule-site.png)
![](/TP5-190226/Images/Partie6/site-rule-general.png)

Résultat :

![](/TP5-190226/Images/Partie6/site-autorise.png)

- Questions :

1. Filtrer par IP source ?
> C'est pour éviter que n'importe qui accède au site. C'est donc pour la sécurité.

2. Filtrer par port ?
> C'est un site web, il n'y a pas besoin que d'autres ports soient ouvert autre que le 80.

3. Pourquoi le pare-feu protège-t-il le LAN même en réseau interne ?
> On n'est jamais à l'abri d'une propagation d'un virus. Si quelqu'un branche une clé usb infectée, il risque moins d'infecter les autres machines. De plus, c'est une garantie que quelqu'un en interne ne casse pas tout en faisant une fausse manœuvre.

### 13. Logs et analyse

On active l'option pour voir les logs :

![](/TP5-190226/Images/Partie6/logs-jeux-argent.png)

![](/TP5-190226/Images/Partie6/journalisation-active.png)

- Sur la VM1 :

``` bash
# Tenter de se connecter à winamax
wget http://winamax.fr
```

![](/TP5-190226/Images/Partie6/ping-winamax.png)

![](/TP5-190226/Images/Partie6/jeux-argent-bloque.png)


- Questions :

1. Différence entre paquet bloqué et autorisé
> Dans les logs, les paquets bloqués sont avec une croix rouge et ne sont pas autorisés à passer pfsense tandis que les paquets autorisés sont avec une coche verte et peuvent passer.

2. Identifier quelle règle a déclenché le blocage
> Il y a une colonne rule qui permet de voir la règle. Par exemple, dans l'image ci-dessous, c'est la règle "Jeux d'argent" qui a bloqué les paquets.
![](/TP5-190226/Images/Partie6/jeux-argent-bloque.png)

3. Comprendre le sens du trafic
> Il y a les colonnes source et destination qui permettent de voir le sens. De plus, avec l'interface, on peut également savoir le sens. Si l'interface est LAN, alors les paquets viennent des VMs locales.


### 14. DMZ

- Questions :

1. Qu'est ce qu'une DMZ ?
> Une DMZ (zone démilitarisée) est un morceau du réseau qui est accessible depuis l'extérieur. C'est une zone isolée du reste du réseau qui permet aux entreprises qui ont un site web, par exemple, d'éviter que les visiteurs malveillants, aillent sur leur réseau interne. C'est donc une sorte de réseau public dans un réseau privé.

2. Pourquoi isoler un serveur ?
> Pour éviter qu'un visiteur malveillant ne se balade dans le réseau privé de l'entreprise où passe, éventuellement, des données sensibles. C'est une mesure de sécurité.

3. Une machine en DMZ peut-elle accéder au LAN ?
> Non, car la machine dans la DMZ est exposée. Donc, si elle se fait infecter et qu'elle est liée au LAN, elle va contaminer tout le monde. Donc il vaut mieux éviter même si c'est possible.

4. Le LAN peut-il accéder librement à la DMZ ?
> Oui, si on reprend l'exemple du site web, il faut que les développeurs puissent mettre à jour le site web donc il vont connecter leur PC (sur le LAN) au serveur (dans la DMZ).


### 15. Filtrage MAC

On regarde l'adresse MAC de la VM1 Ubuntu :

![](/TP5-190226/Images/Partie6/MAC-adresse.png)

On va sur Services > DHCP Server > LAN > Edit Static Mapping

![](/TP5-190226/Images/Partie6/Mac-configure.png)

![](/TP5-190226/Images/Partie6/mac-dhcp.png)


- Question :

1. Le filtrage MAC est-il réellement sécurisé ?
> Non, il n'est pas robuste, il permet de faire en sorte d'avoir toujours la même IP sur le même équipement. Par exemple, pour une imprimante dans une entreprise, cela permet de ne pas avoir à chercher l'IP tout le temps.

2. Pourquoi est-il facilement contournable ?
> L'adresse MAC apparaît en clair lors des échanges ce qui n'est pas sécurisé. Donc, un pirate avec un logiciel comme Wireshark peut faire une attaque contre un matériel précis sans difficulté. Enfin, l'adresse mac est "unique" donc si un pirate l'a, c'est un peu comme s'il connaissait la carte d'identité de la carte réseau et donc de l'ordinateur.


### 16. Portail captif

On configure le portail dans Services > Captive Portal:

![](/TP5-190226/Images/Partie6/Captive_Portal-name.png)
![](/TP5-190226/Images/Partie6/Captive_Portal-config1.png)
![](/TP5-190226/Images/Partie6/Captive_Portal-config2.png)

Résultat :

![](/TP5-190226/Images/Partie6/captive_portal-result.png)

Je peux ensuite utiliser internet :

![](/TP5-190226/Images/Partie6/captive_portal-result1.png)

- Questions :

1. Dans quels contextes utilise-t-on cela ?
> Sur du wifi public, car ça permet à l'utilisateur de connaître toutes les règles à suivre pour utiliser le wifi (Conditions Générales d'utilisation). Ça permet aussi d'authentifier les utilisateurs pour leur donner des droits différents (débit).

2. Quelle(s) avantage(s) avec une simple règle de pare-feu ?
> Contrairement aux règles, le portail captif permet de mieux contrôler les utilisateurs comme les obliger à accepter les CSG (Conditions Générales d'utilisation) ou alors à les obliger à se connecter à un compte par exemple. Il est donc plus facile de gérer les utilisateurs contrairement aux règles où c'est plus l'anarchie.

### 17. Sauvegarde / restauration

Pour sauvegarder, on va dans Diagnostics > Backup & Restore > Backup & Restore :

![](/TP5-190226/Images/Partie6/sauvegarde.png)

On modifie les configs :

![](/TP5-190226/Images/Partie6/sauvegarde-modif1.png)
![](/TP5-190226/Images/Partie6/sauvegarde-modif2.png)

On voit bien que les modifications ont été effectuées :
![](/TP5-190226/Images/Partie6/sauvegarde-modif2-result.png)

Pour restorer, on retourne dans Diagnostics > Backup & Restore > Backup & Restore :
![](/TP5-190226/Images/Partie6/sauvegarde-restore.png)

Résultat :

![](/TP5-190226/Images/Partie6/sauvegarde-restore1.png)
![](/TP5-190226/Images/Partie6/sauvegarde-restore2.png)


- Question :

1. Pourquoi la sauvegarde régulière est-elle essentielle en production ?
> Un bug ou un virus est vite arrivé et, dans ces moments-là, il faut réussir à remettre tout en place le plus vite possible donc il vaut mieux avoir une sauvegarde à charger en quelques secondes plutôt qu'un service non-fonctionnel pendant des heures.



