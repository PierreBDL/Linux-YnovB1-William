## **TP 2 - Vendredi 26 Février 2026** <br>
**Fais sur une VM Ubuntu Server** <br>
**Pierre BDL**

---

## Configuration

### Installation de la VM

Création du compte Ubuntu :

![](./Images/Intro/profil.png)

Activer le SSH :

![](./Images/Intro/ssh.png)

Installation de openvpn et easy-rsa :

``` bash
# Mise à jour des paquets
sudo apt update -y
sudo apt upgrade -y

# Installation
sudo apt install openvpn -y ; sudo apt install easy-rsa -y
```

![](./Images/Intro/install.png)

---

### Se connecter en SSH

On active la redirection de port sur VirtualBox

![](./Images/Intro/redirection-port.png)

On se connecte en SSH :

``` bash
# Connection en ssh
ssh -p 2222 pierre@127.0.0.1
```

## Partie 1 : Comprendre la PKI

- Questions

1. À quoi sert une autorité de certification (CA) ?
> IL permet d'identifier un service ou une entreprise. C'est comme une carte d'identité qui permet au destinataire de confirmer que le service est officiel et donc, normalement, sécurisé.

2. Quelle différence entre clé privée et certificat ?
> Une clé privée est, comme son nom l'indique, privée et donc, elle ne doit pas être divulguée à quiconque tandis qu'un certificat a pour but d'être divulgué lors de l'installation du service qu'on propose donc c'est publique.

3. Pourquoi un serveur VPN a-t-il besoin de certificats ?
> Pour que le client puisse se sentir en sécurité, le VPN doit avoir un certificat à montrer à ce dernier. De plus, le certificat a également pour but d'identifier les clients légaux qui peuvent fournir le certificat pour montrer qu'il pont une license et les autres, qui ne l'ont pas.

### Création de l'infrastructure Easy-RSA

- Environnement :

``` bash
# Création du dossier pki qui va stocker le certificat
mkdir pki && cd pki
```

- Génération :

    - La CA

    ``` bash       
    # Génération de la clé de la CA
    sudo openssl genrsa -aes256 -out ca.key 4096

    # Création d'un nouveau certificat public de la CA à partir de zéro, sans mot de passe pour une durée de 10 ans
    openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt
    ```

    - Certificat serveur

    ``` bash  
    # Créer création clé 
    openssl genrsa -out server.key 2048

    # Création de demande de signature
    sudo openssl req -new -key /etc/openvpn/server/server.key -out ~/server.cs

    # Signer le certificat avec la clé CA  
    openssl x509 -req -in ~/server.csr \ -CA /etc/openvpn/server/ca.crt \ -CAkey /home/pierre/pki/ca.key \ -CAcreateserial -CAserial ~/server.srl \ -out ~/server.crt -days 365 \ -extfile ~/server_ext.cnf


    # Copier dans les dossers d'OpenVPN
    sudo cp ~/server.crt /etc/openvpn/server/server.crt
    ```

    - Certificat client

    ``` bash  
    # Créer création clé  privée
    openssl genrsa -out client.key 2048

    # Création de demande de signature
    openssl req -new -key client.key -out client.csr

    # Signer le certificat avec la clé CA
    openssl x509 -req -in client.csr \ -CA /etc/openvpn/server/ca.crt \ -CAkey /home/pierre/pki/ca.key \ -CAcreateserial -CAserial ./client.srl \ -out client.crt -days 365

    # Vérif :
    openssl verify -CAfile /etc/openvpn/server/ca.crt client.crt
    ```

    - Paramètres Diffie-Hellman

    ``` bash  
    # Créer de la clé de chiffrement pour les échanges
    openssl dhparam -out dh.pem 2048
    ```

    - Clé TLS supplémentaire

    ``` bash  
    # Génèration d'une clé aléatoire
    openssl rand -out tls.key 2048
    ```

- Questions :

1. Où Easy-RSA crée-t-il ses fichiers ?
    > Dans le dossier local. Si on a créé et on s'est déplacé dans un dossier, il les créera dans ce dossier.

2. Que contient le dossier pki/ ?
    > Toutes les clés, certificats et autres données qui doivent être protégées, car, avec elles, on peut usurper une entreprise très facilement et on peut littéralement tout faire sur le VPN.

3. Quelle est la différence entre gen-req et sign-req ?
    > Quand on génère avec gen-req, le fichier n'est pas signé par la clée tandis que sign-req l'est.

4. Que se passe-t-il si vous oubliez de signer un certificat ?
    > Si c'est le certificat client, le service comme le VPN ne sera pas reconnu, car le certificat sera faux.

## Partie 2 : Configuration du serveur OpenVPN

### serveur.conf

``` bash  
# On copie l'exemple qu'il y a dans openvpn
cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/

# On modifie server.conf
sudo nano /etc/openvpn/server/server.conf
```

Remplir avec :

![](./Images/Partie2/config-server.png)

- Questions 

1. Que signifie dev tun ?
    > Tun permet d'économiser des octets dans la bande passante en enlevant l'encapsulation ethernet. En effet, la couche d'encapsulation ethernet n'est pas utile.

2. Quelle est la différence entre UDP et TCP pour un VPN ?
    > UDP privilégie la vitesse à l'intégrité des données tandis que TCP fait l'inverse, il demande des confirmations de l'intégrité des données lors de la transmission au détriment de la vitesse. De plus, TCP est la méthode de base à utiliser si UDP est indisponible. Il vaut mieux utiliser UDP pour avoir de meilleures performances lorsqu'on fait un VPN.

3. Quelle plage IP choisir pour le VPN ? Pourquoi ?
    > La plage d'adresse de base d'OpenVPN est 10.8.0.0/24. Mais le but est quand même de rester sur un réseau privé.


### Routage et NAT

``` bash  
# On modifie la configuration pour activer le forwarding
sudo nano /etc/sysctl.conf

# Mettre l'option ou décommenter
net.ipv4.ip_forward = 1

# On recharge la configuration
sudo sysctl -p
```

![](./Images/Partie2/forwarding.png)

``` bash  
# On active la règle qui change l'IP du client (LAN) en IP publique
sudo iptables -t nat -A POSTROUTING -s 10.33.0.0/24 -o enp0s3 -j MASQUERADE

# On autorise le passage par le pare-feu des requêtes du LAN
sudo iptables -A FORWARD -s 10.33.0.0/24 -o enp0s3 -j ACCEPT
sudo iptables -A FORWARD -d 10.33.0.0/24 -i enp0s3 -m state --state RELATED,ESTABLISHED -j ACCEPT

# On sauvegarde les configs
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

- Questions :

1. Où se configure le paramètre ip_forward ?
    > Ca se configure dans /etc/sysctl.conf.

2. Quelle commande permet d'afficher les règles NAT actuelles ?
    > La commande "sudo iptables -t nat -L".

3. Pourquoi faut-il "masquerader" le réseau VPN ?
    > Car ça permet de mettre l'adresse WAN de la VM du pare-feu à la place de l'adresse LAN du pare-feu. Sans ça, l'adresse privée serait sur Internet et ne serait pas comprise, car lors de la réponse de la requête, cette dernière va chercher l'adresse, mais ne va pas la trouver.

### Démarrage et analyse du service

``` bash  
# Déplacer toutes les clés et autres certificats dans /etc/openvpn/server/
sudo mv /home/pierre/pki/tls.key /etc/openvpn/server/
sudo mv /home/pierre/pki/ca.crt /etc/openvpn/server/
sudo mv /home/pierre/pki/server.crt /etc/openvpn/server/
sudo mv /home/pierre/pki/server.key /etc/openvpn/server/
sudo mv /home/pierre/pki/dh.pem /etc/openvpn/server/

# Donner les droits à OpenVPN
sudo chown root:root /etc/openvpn/server/*
sudo chmod 600 /etc/openvpn/server/server.key

# Modifier le fichier /etc/openvpn/server/server.conf
```

![](./Images/Partie2/config-modif.png)

```
# Démarrage du VPN avec la config qu'on a rédigé
sudo openvpn --config /etc/openvpn/server/server.conf

# Voir l'état du service
sudo sytemctl status openvpn
```

![](./Images/Partie2/status.png)

- Si le service échoue :

1. Quelle commande permet d'afficher les logs système d'un service ?
> La commande "sudo journalctl -xeu openvpn-server@server" pour afficher les derniers logs avec des explications du service openvpn.

2. Quelle est la différence entre status et journalctl ?
> Status permet de voir, du premier coup d'oeil, l'état du service qu'on recherche tandis que journalctl permet d'avoir des informations détaillées.

3. Les chemins vers les certificats sont-ils corrects ?
> Pour qu'OpenVPN puisse accéder aux clés et aux certificats, il faut qu'il ait les droits donc il faut que les fichiers soient dans son dossier /etc/openvpn/server/

## Partie 3 : Création du profil client

``` bash  
# On copie l'exemple qu'il y a dans openvpn
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client/

# Je copie les fichiers client dans les dossier de openvpn
sudo cp client.crt /etc/openvpn/client/
sudo cp client.key /etc/openvpn/client/

# On modifie client.conf
sudo nano /etc/openvpn/client/client.conf
```

client.conf :

``` bash
# -- Partie config
client
dev tun
proto udp

# -- Partie IP
remote 192.168.68.51 1194

resolv-retry infinite
nobind
persist-key
persist-tun

# -- Certificats --
<ca>
-----BEGIN CERTIFICATE-----
MIIFazCCA1OgAwIBAgIUcpBxxH5pPKm/r3eMDzT77v2ys84wDQYJKoZIhvcNAQEL
BQAwRTELMAkGA1UEBhMCRlIxEzARBgNVBAgMClNvbWUtU3RhdGUxITAfBgNVBAoM
GEludGVybmV0IFdpZGdpdHMgUHR5IEx0ZDAeFw0yNjAyMjYxNTQ4MTJaFw0zNjAy
MjQxNTQ4MTJaMEUxCzAJBgNVBAYTAkZSMRMwEQYDVQQIDApTb21lLVN0YXRlMSEw
HwYDVQQKDBhJbnRlcm5ldCBXaWRnaXRzIFB0eSBMdGQwggIiMA0GCSqGSIb3DQEB
AQUAA4ICDwAwggIKAoICAQDazyyPF9JFYcUngBcSycKq8c6PUPKbWu4Vfu+V69Bx
/8dVqLFNCdgyAnHqaZ1IGfcZfDpkwJ2UNZO8Ws2F3eXXjyb+ECXs9VyxXoKnNLHI
6nwXgAgwvZWoqR8OPK3IwuNDun9dQoYseJyBL/i92DSbBP7e76ndKkSmeSihTE88
vl1E+QSe2Du3xfBATQ63MOk908JrS5IADTM9S7BEnBn70ZFzYX8tzHxTKg+mbxeT
giQpByzC/dJkXAlfLw15/uQ+wmVftK6CpaPtp2Ku21H0sClg+6KqYlxcyUcWB/It
dNsi+0xvcrmRLydxRdkbn0AIhTwWHiSWSLIjSx4WSB+Bx1CMzqJEysitbGkyaC+X
ZMtp0Ha85OXaxyOwExERWdbbVLX8ytN3Uyf+UN8W1Az/xkfRmVobL43kavY6RBPb
ObizNCUS6Vgw+UhBN//9f8O7FrpVMPB6iW1DHKbnLeU9cZDsptrTDnCTDuO6QSJJ
8XSxO/tf8x/XWHfuXawlO/UZbTiGJLFZRQNyYDnkN0KYxlQLm/AExdizVD0Pw+eO
tNqKAJSlPWpxZsmjP8TtBf6O3Kl1RqG47i0duwdFPJScJ5gtv0ZAQmjrYldj7MxJ
kYqPG01Vfrh/6uxlPVZQz4ra1rH37bizdfYJGbECC1QuY/d20olxCODaWyZoHsbs
IQIDAQABo1MwUTAdBgNVHQ4EFgQU7rKzOltsXpKp+ZEWXXqvwLWnC4EwHwYDVR0j
BBgwFoAU7rKzOltsXpKp+ZEWXXqvwLWnC4EwDwYDVR0TAQH/BAUwAwEB/zANBgkq
hkiG9w0BAQsFAAOCAgEAmabQuztlO1eCut2YYBLWkg8HKHU2LJcSkfaLiySTNEJx
P2QbkjNyAMJ85kDfW7tYqMcG10jPxLtYjfk+/3WFaiXV5bTiUfLpqROVLEBUcxQf
XGCgGRMiBFqhhN1eYkffH37fPBcjtdu16hO2f3DjBLErVIC1/cXljZCv37zRnO8z
SlxxBF/UDtIom/K5pB1/G8M4IDltH6hPqS6RdsVYS9+wQIU/6fj7LXNE4c8HWXFS
pSwNt9CE2+yih9yLtnv+RpCGkfzZvypf6JD82Bx+jFrLa3WbFdgUeyH189w8K++1
FYorvpHuS0kUa8Wq11MjtLwpaGW66UcyBR7Y2m7orxLCzoU7+uwTEP2/OeNi0Vuk
xOI3K7/6P2lnJ5/UffPG21YEtmhnNkIOkgA8VypMuiocN9JdbTBqdgPZHisoeWSU
zZgCjN+HlFaREEyQ5+2bnN4QR7C8rqqMfEBotgsWptKPlh9+hiB61sca8RGJk5EO
Qfuj4RX2x6TzOHjDii8TgQc25KYqcLahhQFVce6mwZwrs8UNDuLvp+dyppBJYqYh
B/byTnat/10XzA9iNRbR5X+QUL7qn0rSr+YPgEStK9JHqkf/GenazOeY4Yz6izZg
m6MCZwYVZeEtr6ccbXM6JEnQxXMPWa/52v/jL2fny62/DMpHtHZKwzytDU9aF9M=
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
MIID3jCCAcYCFDsBt8AtTI7spESybW4+9XlnFtSiMA0GCSqGSIb3DQEBCwUAMEUx
CzAJBgNVBAYTAkZSMRMwEQYDVQQIDApTb21lLVN0YXRlMSEwHwYDVQQKDBhJbnRl
cm5ldCBXaWRnaXRzIFB0eSBMdGQwHhcNMjYwMjI4MTIxMDUwWhcNMjcwMjI4MTIx
MDUwWjASMRAwDgYDVQQDDAdjbGllbnQxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEA2yfDCKwO9qJOA9ls8tP/en4Yn1dK7WkoVALWXtrUWAqyuIQ5iJGz
RVEyzN7OpJbQCjYeUQuubJFy24XIQiaqFBKomF1fybCGPwXLyMWOx3TtK8j48wPg
qq7pWtorq+b1T6N493hhkIluhodkF/cCAEq+n5xC2RCOWFLtqQZKXFEHSI2qM9bD
uz4MRwE7xH+FUE9JPGWf2+WCyGbt6XigFrt5LeNcK4d2YLmAEGdsJ854P1BbGij8
MlGvjADAwTTMd1+Wyiueity8iJl0f9IYL4pCv3csqTlr7+A4vKtZkebDmGtK6/dc
HnTFl0BTTkLvm21E3w23jl+0woBcktIUxQIDAQABMA0GCSqGSIb3DQEBCwUAA4IC
AQCrVkD3AUAcUN2TxYhgRMz3Qz8DsRZX7RFRkJDp/qznaKIsUGixgRcpLC8QuVqS
F4RNiEyvnMTdw+LuB5YWCergGvdcrJRkd4wuMwozKSxZkLjHRATHkjC+hEkArOK+
3S4JpYpM1s+9St1uygOyakpSOWHE4Bx8+JocVymm51Q88ms3PxrqUC+/O2bwZdFE
dKX1WdIwnjKE7C5rW+XEyH322rHXPfnO8ZEBGYmbBr7eRBgySpBRXgHjfDkqd8Xq
FGg5ec3XRogzizXSHGFZ5A6f2Dh0TMv5rWQ138RQawIKgiYPoohZjUWfF3axXp9N
DWxjc+iLmMl8IGpr1JlEtfNFh5B3s1F3jCLkOiS1dphDsf5f2gd6S7xxxUVKZBcu
+aNI58BsxLjkQq7fwUnIOsYZHgRRohUpmUYTqgwkoh/Gm3BXkMjSMb1teZyE6Hcg
UeFrTCOxIBkzOOdp8dasuASC5EN1sEtOH2Nd7EpZH/mAQ8V+fyk0OfL+CwxtjiVf
cuJrqMUD6buP0JijUlckoXJ0K32Oc9FlXS3w4o+2COTBPIgCJMxc6JH/jw7F/EmB
fxVrDcumQWachgvlR0oIr+WNLBXw0CD5lUHuJENgkcAwUVoidlljx6flstSiAXr7
hdwpoSfCSlw0gtfCIPAa3PBTL+CQv6KnHmVqpxSkjAVwBQ==
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDbJ8MIrA72ok4D
2Wzy0/96fhifV0rtaShUAtZe2tRYCrK4hDmIkbNFUTLM3s6kltAKNh5RC65skXLb
hchCJqoUEqiYXV/JsIY/BcvIxY7HdO0ryPjzA+Cqrula2iur5vVPo3j3eGGQiW6G
h2QX9wIASr6fnELZEI5YUu2pBkpcUQdIjaoz1sO7PgxHATvEf4VQT0k8ZZ/b5YLI
Zu3peKAWu3kt41wrh3ZguYAQZ2wnzng/UFsaKPwyUa+MAMDBNMx3X5bKK56K3LyI
mXR/0hgvikK/dyypOWvv4Di8q1mR5sOYa0rr91wedMWXQFNOQu+bbUTfDbeOX7TC
gFyS0hTFAgMBAAECggEAEHX3mb6mrxK4zmQ14TVYMqCnA8zgNV3ZKBpAdMZ1/Zjs
HJzd32qmLvYu3SF9HvKu9J1lAIxikQgJuwBc7s8Npk1SeYNF7xXBt5kdWdqw4z1e
lRPW0xX++YMCxw6tCg/ibBVowBm8JkmUUil4NK6aaWAz2jQ39yrdrjeoXzI4Qp+1
vrus1vFyui7JdFnvX6Df2lsPkXfH3RU81sux96SKEdvvCHqcowBgumtOclyLhmm3
MnzeZ6lMNIn1gai6xD8j3MHfoq4SX7rsn7CLNmV1hui/TfZoo+YKijuz6GYXYrrj
efzuUj3i5eqgFqc9uKuDY/77MsVvyA/ehqTEMZi73wKBgQD/2IM3qSZA/1eZASpV
H/NBo2h2IowFsYSyQizGCE7jWWfCkETX18ysYaZHBhUGNSwG6ecuW9ajVK5GA5U5
JpMNJkmHlk+EBcLKuKkxcufWexYkTIhTFy1pzE8FWN/SwTq5WTR5yi1lEjP2tTgj
fHA3y1/aZF4oy5BnGktx/veXrwKBgQDbSZYhy8/p2vJ04LgnVcWTd37yRvJ4Smie
mMP7+AT1OTSROlUAEdZQd/zKBkq1DR5/XV1FXeRPJi2kXyQTwv5QAMbIG4KZKkHX
QdliYyHa6/SxYZqYPB+Xgx2iyBa9F8AGj5gn/8rKM+6+BLqZqBoehjXd13Um7k4W
8iuQTVZDywKBgQDSN7uxi2mkNGpsjLmU0jjVMQhPMRmuP+tRcPBLE16Gjdqk0lfO
pkFFTsMKTqQNwrjXQW+z9sAI2jaLzrvZBqoDPvdh0g6p4vXoDgzvVYPLBr3tu3i0
FTI6YSZ4tUrLZ/vxeauVepsMFgXea46iEcg+//tVfeykRobzQHbHRwF3fwKBgAfC
Z9SUUKwWukOWfA2wREh+jvhluEi3YVJTV4P/97TxrEGDTeStvOamCEThyhzz9ONO
23EL7Hs8f5r0XuNwH+2WyXHlP07KJwBF7TpFBbqoZMUuDflbD0ceNZs/90YDpCDX
TewEahmTktwYcL3lK99uolNSP2v8PLklIpN9mRLXAoGBAJ5DEIjNzjfUdN/ckUFs
PVhP/NLgE9DM8IJTnT5Zj3lrcXApvefZRIX0QldSp7/POPdOgc5h606xu/wKsqoM
q4IKnohWHX7GCUTiYg5SYId1XEAw1FBOu/SSrwK0upP3PSjh+8KuKPgkclmYwen1
Q1Nb6IlDrP3pITkQpGXQHZtP
-----END PRIVATE KEY-----
</key>

<tls-crypt>
#
# 2048 bit OpenVPN static key
#
-----BEGIN OpenVPN Static key V1-----
c40598badf04143f48edb70f54cfd90c
39fbb8ff535de27727915adc8e8333c8
15729c3ebf9b1a308545d7f25e52a28c
0e063912a6a6248bb280e4e5ecd9e137
98fbc20e9a8b93a2ab2b88abb27f1ecd
ce176bb9e81c59ed8bdeba1240b9fb42
fc7acc218219954a429dfabc71259f3e
a6a67352f0dd2f02e25cb8355b435648
46b44ea3a3957b8b139710210cb46bd6
de756e3e5d1d1e64290e7e58e09dc996
6bd656c9279f78a264e5f4e657d58125
6c3a121533fab0be40587cd8d8fc511d
f7a750bd02a1a629c99b6b808101e1dd
179f415eb5b6b6d54511440cd44b058b
686537b3e10c576c8601fb3a373d7751
288b7fcc7bfdd294c1c524da61efdd24
-----END OpenVPN Static key V1-----
</tls-crypt>

remote-cert-tls server

cipher AES-256-GCM
auth SHA256
verb 3
```

- Questions :

1. Comment intégrer un certificat directement dans un fichier .ovpn ?
> En utilisant des balises comme ```<tls-auth> </tls-auth>``` ou ```<tls-crypt> </tls-crypt>```

2. Pourquoi la clé privée ne doit-elle jamais être partagée publiquement ?
> Avec la clé privée, il est possible d'usurper l'identité d'un client, de se faire passer pour lui, etc.


### Tests et validation

On redirige les ports dans VirtualBox > Configuration > Réseau

![](./Images/Partie3/redirection-port.png)

On autorise l'UDP :
``` bash
# On autorise l'UDP sur le port 1194
sudo ufw allow 1194/udp

# On démarre bien le service
sudo systemctl restart openvpn-server@server
```

On convertit le fichier :
``` bash
# convertion
sudo cp client.conf ./client.opvn
```

On extrait le fichier de la VM pour le récupérer sur Windows :
``` bash
# Extraction avec SCP
scp -P 2222 pierre@127.0.0.1:/etc/openvpn/client/client.ovpn "C:\Users\Pierre DE LAVERGNE\Desktop\client.ovpn"
```

On tente de se connecter :

![](./Images/Partie3/connecte.png)

J'ai fais plusieurs heures de tests mais je n'arrive pas à faire en sorte qu'on ait de la connexion sur le client. Le client arrive à ping le server vpn et le vpn arrive à ping le dns et internet mais la connexion est bloquée entre les deux. Parfois, des ping passent mais c'est que un ou deux paquets max. J'ai essayé en déctivant McAfee et en coupant le pare-feu mais sans succès.

Test qui ont échoué :
- Désactiver les pare-feu
- Désactiver McAfee
- Vérification nftables
- Vidage total des règles iptables
- Désactivation DCO
- Priorisation interface VPN
- MTU côté client (taille des paquets)
- Vérification des configs serveur et client
- Autres tests
- ping 8.8.8.8 depuis le client (Détecté sur le serveur mais pas de réponse)