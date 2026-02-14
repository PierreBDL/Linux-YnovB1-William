<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    line-height: 1.6;
    max-width: 1000px;
    margin: 0 auto;
    padding: 20px;
    background: #f5f5f5;
  }
  
  h1 {
    color: #2c3e50;
    border-bottom: 4px solid #3498db;
    padding-bottom: 10px;
    margin-top: 30px;
  }
  
  h2 {
    color: #34495e;
    border-left: 5px solid #3498db;
    padding-left: 15px;
    margin-top: 25px;
    background: linear-gradient(to right, #ecf0f1, transparent);
    padding: 10px 15px;
  }
  
  h3 {
    color: #16a085;
    margin-top: 20px;
  }
  
  h4 {
    color: #7f8c8d;
    font-style: italic;
  }

  p {
    color: black;
  }
  
  code {
    background: #2c3e50;
    color: #ecf0f1;
    padding: 2px 6px;
    border-radius: 3px;
    font-family: 'Courier New', monospace;
  }
  
  pre {
    background: #2c3e50;
    color: #ecf0f1;
    padding: 15px;
    border-radius: 5px;
    overflow-x: auto;
    border-left: 4px solid #3498db;
  }
  
  pre code {
    background: transparent;
    padding: 0;
  }

  .titre p {
    background: transparent;
    font-size: 20px;
    font-weight: 700;
  }
  
  .resultat {
    background: #d5f4e6;
    border: 2px solid #27ae60;
    padding: 10px;
    border-radius: 5px;
    margin: 10px 0;
    font-weight: bold;
  }
  
  .commande {
    background: #fff3cd;
    border-left: 4px solid #f39c12;
    padding: 10px;
    margin: 10px 0;
    border-radius: 0 5px 5px 0;
  }
  
  .note {
    background: #e8f4f8;
    border: 2px dashed #3498db;
    padding: 15px;
    border-radius: 5px;
    margin: 15px 0;
    text-align: center;
  }
  
  img {
    max-width: 100%;
    border: 2px solid #bdc3c7;
    border-radius: 5px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    margin: 10px 0;
  }
</style>

<div class="note titre">

TP 2 - Vendredi 23 Janvier 2026 <br>
Fais sur une VM Fedora

</div>

## I] Exploration en solo

### 1. Base64

Création du fichier file_bin

``` bash
# Créer le fichier de 50 * 1ko soit 50ko
dd if=/dev/urandom of=file_bin bs=1k count=50
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/file_bin-create.png)

Encodage en base64

``` bash
# Encoder le fichier en base 64
openssl base64 -e -in file_bin -out file_bin_b64

# Lire le fichier
cat file_bin_b64
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/file_bin_b64-cat.png)


``` bash
# Comparer les tailles des fichiers commençant par file_bin
ls -l file_bin*
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/difference_taille.png)

On peut voir que le fichier encodé est plus lourd que le fichier de base d'environ 1/3.

``` bash
# Décoder le fichier
openssl base64 -d -in file_bin_b64 -out file_bin2

# Comparaison de la taille (-s pour afficher un message quand c'est identique)
diff -s file_bin file_bin2
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/decodage.png)

On peut voir que les fichier file_bin et file_bin2 sont identiques.


### 1. AES (Chiffrement symétrique)

``` bash
# Création du fichier message avec les mots contenant "ker"
cat /usr/share/dict/words | grep ker  | tr "\n" " " >message

# grep ker : filtre les lignes contenant "ker"
# tr "\n" " " : remplace les sauts de ligne par des espaces
# message : redirige le résultat vers un fichier nommé "message"


# Afficher le contenu du fichier
cat message
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/message-create.png)

``` bash
# Encoder le fichier message
openssl enc -e -salt -in message -out message_c -aes256 -pbkdf2 -md sha256

# enc : sous-commande pour le chiffrement/déchiffrement
# -e : indique une opération de chiffrement
# -salt : ajoute un sel aléatoire pour renforcer la sécurité (on verra ça juste après)
# -in message : fichier d'entrée non chiffré
# -out message_c : fichier de sortie chiffré
# -aes256 : algorithme de chiffrement AES avec une clé de 256 bits
# -pbkdf2 : utilise l'algorithme PBKDF2 pour dériver la clé de chiffrement à partir du mot de passe
# -md sha256 : utilise SHA-256 comme fonction de hachage pour PBKDF2
```

Décodage du fichier

``` bash
# Décoder le fichier
openssl enc -d -in message_c -aes256 -pbkdf2 -md sha256

# Affichage du contenu du fichier
cat message_c
```

Encrodage en base64

``` bash
# Décoder le fichier
openssl enc -e -a -salt -in message -out message_c2 -aes256 -pbkdf2 -md sha256

# Affichage du contenu du fichier
cat message_c2
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/cat_message_c2.png)

### 1. RSA (Chiffrement asymétrique)

Génération une paire de clés RSA

``` bash
# Génération une paire de clés RSA
openssl genrsa -out cle_ynov.pem 2048

# genrsa : génère une paire de clés RSA
# -out cle_ynov.pem : fichier de sortie pour la paire de clés
# 2048 : taille de la clé en bits (plus le nombre est élevé, plus la sécurité est forte, mais aussi plus les opérations sont lentes)


# Affichage du contenu du fichier
cat cle_ynov.pem
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1//C-cle_ynov.png)

``` bash
openssl rsa -in cle_ynov.pem -text -noout
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/C-cle_ynov-noout.png)


``` bash
# Encoder le fichier de la clé privé
openssl rsa -in cle_ynov.pem -aes256 -out cle_ynov.pem

# Exporter la clé publique
openssl rsa -in cle_ynov.pem -pubout -out clepublique_ynov.pem

# Afficher la clé
cat clepublique_ynov.pem
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/C-cle_publique.png)

``` bash
# Visualiser les paramètres
openssl rsa -in clepublique_ynov.pem -pubin -text -noout

# -pubin : indique que le fichier d'entrée contient une clé publique et non une clé privée
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/C-modulo-cle_publique.png)

<div class="note">
<strong> Observation </strong> On peut voir que c'est une clé RSA de 2048 bit donc modulo 2048 et que l'exposant est de 65537 qui permet e vérifier la clé.
</div>

``` bash
# On créer un fichier qui va être encodé
sudo nano pass_ynov

# On l'encrypte avec la clé qu'on a créé
openssl pkeyutl -encrypt -in pass_ynov -inkey clepublique_ynov.pem -pubin -out pass_ynov_c

# pkeyutl : utilitaire pour les opérations cryptographiques de base avec des clés
# -encrypt : chiffre le contenu du fichier d'entrée
# -in pass_ynov : fichier contenant la donnée à chiffrer
# -inkey clepublique_ynov.pem : clé publique utilisée pour le chiffrement
# -pubin : indique que la clé d'entrée est une clé publique
# -out pass_ynov_c : fichier de sortie pour les données chiffrées
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part1/C-chiffrer-cle.png)

``` bash
# On décode le fichier dans pass_ynov_c
openssl pkeyutl -decrypt -in pass_ynov_c -inkey cle_ynov.pem

# -decrypt : déchiffre les données
# -inkey cle_ynov.pem : utilise la clé privée correspondante pour le déchiffrement
```

## II] Sans que je vous file les réponses à chaques étapes

### A) Base64

#### 1. Génération d’un fichier binaire

``` bash
# Créer le fichier de 100 * 1ko soit 100ko
dd if=/dev/urandom of=data.bin bs=50k count=2

# Afficher la taille des fichiers
ls -l
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap1/creation_fichier.png)

#### 2. Encodage

``` bash
# Encoder le fichier en base 64
openssl base64 -e -in data.bin -out data.b64

# Lire le fichier
cat data.b64
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap1/2-encoder64.png)

``` bash
# Regarder la taille
ls -l
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap1/2-encoder64-taille.png)

On peut voir que le fichier encoder fait environ 1/3 de poids en plus que le fichier source

#### 3. Décodage

``` bash
# Décoder le fichier
openssl base64 -d -in data.b64 -out data_restored.bin

# Comparaison de la taille (-s pour afficher un message quand c'est identique)
diff -s data.bin data_restored.bin
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap1/3-diff.png)

#### 4. Questions

1. Base64 est-il un chiffrement ? Pourquoi ?
> Base 64 encode les fichiers et ne les chiffre pas. On a pu voir qu'il fallait une clé pour chiffrer et déchiffrer une fichier; or, base64 n'en aavait pas donc c'est de l'encodage. Base64 permet de transformer le contenu du fichier en caratères ASCII.

2. Pourquoi la taille du fichier change-t-elle après encodage ?
> Si j'ai bien compris, lorsqu'on encode 3 octets de données, on le fait sur 4 octets. Avec cette augmentation, on obtient +1/3 de taille de fichier.

3. Quel est approximativement le pourcentage d’augmentation ?
> Le pourcentage d'augmentation de la taille est d'environ 1/3.

4. Quelle méthode permet de vérifier rigoureusement que deux fichiers sont identiques ?
> On utilise la commande `diff -s fichier1 fichier2`

### B) Chiffrement symétrique – AES

#### 1. Création d’un message

``` bash
# Créer un fichier confidentiel.txt
sudo nano confidentiel.txt
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap2/confidentiel-txt.png)

#### 2. Chiffrement

``` bash
# Encoder le fichier
openssl enc -e -salt -in confidentiel.txt -out confidentiel.enc -aes256 -pbkdf2 -md sha256

# Affichage du contenu
cat confidentiel.enc
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap2/cat-confidentiel-encode.png)


#### 3. Déchiffrement

``` bash
# Décoder le fichier
openssl enc -d -salt -in confidentiel.enc -out confidentiel2.txt -aes256 -pbkdf2 -md sha256

# Affichage du contenu
cat confidentiel.enc
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap2/confidentiel-decode.png)

<div class="note">
<strong> Observation </strong> On peut voir que le contenu est identique à ce qui est dans confidentiel.txt.
</div>

#### 4. Analyse

``` bash
# Encodage du fichier
openssl enc -e -salt -in confidentiel.txt -out confidentiel2.enc -aes256 -pbkdf2 -md sha256

# Véridier les différences
diff -s confidentiel.enc confidentiel2.enc
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap2/rechifrage-diff.png)

<div class="note">
<strong> Observation </strong> On peut voir que le chiffrement diffère même avec le même mot de passe. Donc lors du chiffrement, les clé sont uniques via le salt et PBKDF2.
</div>

#### 5. Questions

1. Pourquoi les deux fichiers chiffrés sont-ils différents ?
> Je pense que c'est pour éviter que quelqu'un puisse accéder au fichier s'il a la même clé. Cette méthode permet que chaque chiffrement soit unique et donc plus sécurisé.

2. Quel est le rôle du sel ?
> Le sel permet de rendre chaque chiffrement unique et donc d'améliorer la sécurité. De plus, ça permet que le chiffrement diffère même avec le même mot de passe.

3. Que se passe-t-il si une option change lors du déchiffrement ?
> Le déchiffrement ne pourra pas fonctionner car le chiffrement répond à un algorithme (via les options). Si on change ces options, l'algorithme différère et le déchiffrement est impossible. Donc le fichier restera inlisible.

4. Pourquoi utilise-t-on PBKDF2 ?
> PBKDF2 permet de mieux sécuriser le fichier en encodant le mot de passe. De plus, si j'ai bien compris, il va faire une dérivation de la clé qui va améliorer la sécurité.

5. Quelle est la différence entre encodage et chiffrement ?
> L'encodage est une transformation des caractères qui n'a pas besoin de clé pour être décoder (il "suffit" d'avoir l'algorithme comme base64). L'encodage est donc un moyen peu sécurisé car facilement contournable si on sait l'algorithme. Le chiffrement utilise une clé qui va transformer de manière unique chaque caractère. Le chifffrement va donner une clé qui est obligatoire si on veut faire l'opération inverse. Le chiffrement est donc une technique beaucoup plus sécuriser le l'encodage.

### C) Cryptographie asymétrique – RSA

#### 1. Génération de clés

``` bash
# Génération la clé privé
openssl genrsa -out rsa_private.pem 2048

# Exporter la clé publique
openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap3/cle_prive-cle_public.png)

``` bash
# Chiffrement de la clé privé
openssl rsa -in rsa_private.pem -aes256 -out rsa_private_p.pem

# Affichage des paramètres de la clé privée
openssl rsa -in rsa_private_p.pem -text -noout
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap3/chiffrement_private.png)

``` bash
# Affichage des paramètres de la clé publique
openssl rsa -in rsa_public.pem -pubin -text -noout
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap3/parametres-public.png)

<div class="note">
<strong> Observation </strong> Dans la clé privée, qui a été chiffré, il y a le modulo, l'exposant publique, l'exposant privé, le coefficient tandis que dans la clé publique, il n'y a que le modulo et l'exposant. La clé publique n'a que pour fonction de chiffrer le données mais ne doit pas faire l'inverse. Au contraire, la clé privée doit pouvoir tout faire (chiffrer, déchiffrer) donc elle a besoin de plus de choses.
</div>

#### 2. Chiffrement asymétrique

``` bash
# Créer le fichier secret.txt avec le texte "Bonjour"
echo "Bonjour" > secret.txt

# Chiffrer avec la clé publique
openssl pkeyutl -encrypt -in secret.txt -inkey rsa_public.pem -pubin -out secret.enc
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap3/cryptage-fichier.png)

``` bash
# On déchiffre le fichier
openssl pkeyutl -decrypt -in secret.enc -out secret2.txt -inkey rsa_private_p.pem

# On affiche le contenu du fichier déchiffré
cat secret2.txt
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap3/dechiffre-fichier.png)

<div class="note">
<strong> Observation </strong> On peut voir que le coontenu a bien été déchiffré et nous donne bien le bon résultat à savoir ("Bonjour")
</div>

#### 3. Questions

1. Pourquoi la clé privée ne doit-elle jamais être partagée ?
> C'est cette dernière qui permet de déchiffrer les données. Donc si jamais on la partage, tout ce qui l'ont pourront déchiffrer les fichiers. Dans le monde réel, c'est comme partager son mot de passe d'ordinateur ou de téléphone, out le monde pas accèder à nos données sensibles. De plus, la clé proivé permet aussi de chiffrer les données, donc ça ouvre la porte à de l'usurpation d'identité.

2. Pourquoi RSA n’est-il pas adapté au chiffrement de gros fichiers ?
> RSA demande beaucoup de calcul à l'ordinateur  donc si on commence à chiffrer de gros fichiers, on peut réduire les performances de l'ordinateur. De plus, RSA a été conçut pour de petits chiffrement et n'est donc pas adapté à de gros fichiers. De plus, RSA a une limite de quelques centaines d'octets pour la taille des chiffrements.

3. Quelles différences observe-t-on entre les paramètres d’une clé publique et d’une clé privée ?
> Comme dit plus haut, la clé privé a accès à tout ce qu'a la clé publique mais également à d'autres paramètres qui permettent à la clé privée de pouvoir déchiffrer les données.

4. Quel est le rôle du modulo dans RSA ?
> Le modulo est la base du modèle RSA, c'est lui qui va vraiment créer la sécurité.

5. Pourquoi utilise-t-on souvent RSA pour chiffrer une clé AES plutôt qu’un document entier ?
> Etant donné que RSA est conçut pour chiffrer de petits fichiers, il est plus facile et rapide de chiffrer une clé qui permettra, elle-même, de chiffrer de plus gros fichiers. Ca permet également d'augmenter la sécurité car la clé est aussi chiffrée.

### D) Signature numérique

#### 1. Création et signature

``` bash
# Créer le fichier contrat.txt avec le texte "Vous êtes engagé"
echo "Vous êtes engagé" > contrat.txt

# Génération du hash
openssl dgst -sha256 contrat.txt

# Signer avec la clé privé
openssl dgst -sha256 -sign rsa_private_p.pem -out contrat.sig contrat.txt
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap3/hash.png)

#### 2. Vérification

``` bash
# Vérifier la signature avec la clé publique
openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap3/verify_public.png)

``` bash
# Modification du fichier contrat.txt
echo "Félicitations" >> contrat.txt

# Vérifier à nouveau la signature avec la clé publique
openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
```

<div class="resultat">
Résultat :
</div>

![](/TP3-130226/Images/part2/chap3/contrat-modif.png)

<div class="note">
<strong> Observation </strong> On peut voir que la signature n'est plus la même car le fichier contrat.txt a été modifié.
</div>

#### 3. Questions

1. Que se passe-t-il après modification du fichier ?
> La vérification de la signature échoue donc la signature est devenue invalide.

2. Pourquoi ?
> Etant donné l'échec entre la première tentative et la deuxième, on peut en déduire que la signature est liée au contenu du fichier. Puisqu'on a moddifier le fichier, la signature a aussi été modifié.

3. Quel est le rôle du hachage dans le mécanisme de signature ?
> Le hashage permet de créer une empreinte unique dépendante du contenu du fichier. Cette technique permet de rendre la signature unique car il faudrait copier le fichier au caratère près ainsi qu'avoir la même clé. Donc le hashage est une partie du procédé qui rend la signature unique. De plus le hashage en lui même ne rend pas le fichier chiffré; c'est la clé privé qui va vraiment le sécuriser.

4. Quelle différence entre signature numérique et chiffrement ?
> La signature est beaucoup utilisé par les grandes marques car elle permet d'assurer une authenticité du signataire. Elle assure aussi le fait que le fichier est complet car si le fichier est modifié, la signature change. On peut donc savoir si le fichier est complet avec une commande. Le chiffrement, lui, a deux clé qui permettent de "vérouiller" le fichier. Le problème est que le chiffrement n'ai, à la base, pas fait pour savoir si le fichier est complet, il se contente de le crypter.