## **TP 2 - Vendredi 23 Janvier 2026** <br>
**Fais sur une VM pfSense** <br>
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
> Pour sécuriser les données qui circulent entre laa VM pfsense et le navigateur web sur l'hôte.

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
> L'interface WAN (Word Area Network) donc l'interface avec l'adresse 10.0.2.15/24.

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
> Comme ça, on n'a pas besoin de configurer les adresses à chaque fois qu'il y a un nouvel appareil. Ca évite aussi les risques de conflit (deux appareils avec la même adresse).

2. Quelle plage d’adresses choisir ?
> Les adresses entre 192.168.56.100/24 et 192.168.56.200/24.

3. Quelles adresses faut-il éviter d’inclure dans la plage ?
> Les adresses telles que : 192.168.56.0, 192.168.56.254, 192.168.56.255 et celle de pfsense (192.168.56.99).

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
> La destination doit être le WAN car les requêtes sont pour des ip externes au LAN. Sur la règle, ce doit être any car on ne sais pas à quoi resesemble le réseau de destination.

3. Faut-il autoriser tous les protocoles ?
> Non, pour le dns, on peut juste autoriser l'UDP. Accepter tout les protocoles peut nuir à la sécurité et augmenter le nombre de failles. Il vaut mieux donner le strict minimum.

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
> Déjà qu'on manque d'adresse IPV4, si on n'avait pas de NAT, le nombre d'appareils pouvant se connecter à Internet serait de 4,3 milliards en comptant les PC, les téléphones, les serveurs, etc. Le NAT est donc nécessaire pour avoir une IPV4, car ça permet à la box, ici pfsense, d'avoir un réseau local avec les appareils de la maison et une adresse publique pour communiquer sur Internet. C'est donc nécessaire pour pouvoir avoir un accès à Internet.

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
> Lorsque l'utilisateur tente de se connecter au site en http, le site affiche "La connexion a été réinitialisé" tandis qu'en https, le site affiche "Impossible de se connecter".
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
> Sinon, on devrait avoir autant de règles que de site qu'on veut bloquer. Donc pour éviter le bazar et le fait que ce soit long et pénible à faire, on fait un alias où on peut juste rajouter le site. En plus, c'est trier par catégories (Exemple : "Jeux-Argent") donc c'est plus facile pour la gestion.

2. Où se créent les alias ?
> On va dans Firewall puis dans Aliases.

3. Comment vérifier qu’une règle bloque réellement le trafic ?
> On va sur une machine du réseau et on tente d'accèder à un site bloquer avec un ping :
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
> Si c'est le cas, "Pass any" est prioritaire et va laisser passer la requête.


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








- Questions :

1. Filtrer par IP source ?
> 

2. Filtrer par port ?
> 

3. Pourquoi le pare-feu protège-t-il le LAN même en réseau interne ?
> 






































