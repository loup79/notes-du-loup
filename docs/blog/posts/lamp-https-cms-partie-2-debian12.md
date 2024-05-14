---
title: "LAMP - VBox/Deb12 2/2"
summary: Création d’un serveur Web sous Linux avec Apache, MariaDB et PHP.
authors: 
  - G.Leloup
date: 2024-04-19
categories: 
  - 8 - LAMP + HTTPS + CMS
---

<figure markdown>
  ![Synoptique - Web PHP MySQL sous Debian 12 - Mémento 8.1](../images/2024/04/web-php-mysql-2-deb12-memento-8.1.webp){ width="430" }
</figure>

## Mémento 8.1 - HTTPS _(SSL/TLS)_

### Protocole HTTPS

Ce protocole servira à garantir la sécurité et l’intégrité des informations échangées entre le site loupvirtuel.fr et les navigateurs Web.

HTTPS combinera le HTTP avec une couche de chiffrement SSL/TLS.

Vous utiliserez, pour le chiffrement, l'outil openssl installé avec Apache.

Les clés et certificats dédiés au HTTPS seront créés dans le dossier /etc/ssl/.

Le domaine loupvirtuel.fr étant fictif, impossible de demander à une Autorité de Certification existante de le valider. Il faudra en créer une localement.

#### _- Certificat PEM (CA)_

CA = Certificate Authority _(Autorité de Certification)_  
PEM = Format de texte codé en Base64

Créez une clé privée contenant un MDP pour le CA :

```bash
[srvdmz@srvdmz:~$] cd /etc/ssl 
[srvdmz@srvdmz:~$] sudo openssl genrsa -aes128 -out loupvirtuel-ca.key 2048 
```

Gérez le retour ainsi :

```markdown
Enter PEM pass phrase: Ce que vous voulez
Verifying - Enter PEM pass phrase:
```

et générez le certificat CA X.509 associé à cette clé :

```bash
[srvdmz@srvdmz:~$] sudo openssl req -x509 -new -nodes -key loupvirtuel-ca.key -sha256 -days 3650 -out loupvirtuel-ca.pem 
```

Gérez le retour pour le **C**ertificate **A**uthority ainsi :

```markdown
Enter pass ... loupvirtuel-ca.key: Votre pass phrase CA
You are about ...
-----
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:Ain
Locality Name (eg, city) []:Bourg-en-Bresse           
Organization Name (eg...) [Internet ... Ltd]:Internet-CA
Organizational Unit Name (eg...) []:Labo Certificats CA
Common Name (e.g... FQDN ...) []:internet-ca.org
Email Address []:Rien -> Touche Entrée
```

#### _- Certificats CSR/CRT_

Créez une clé privée pour le serveur Apache :

```bash
[srvdmz@srvdmz:~$] sudo openssl genrsa -out loupvirtuel.key 2048  
```

et générez le certificat CSR associé à cette clé :

```bash
[srvdmz@srvdmz:~$] sudo openssl req -new -key loupvirtuel.key -out loupvirtuel.csr   
```

Gérez le retour pour le **C**ertificate **S**igning **R**equest ainsi :

```markdown
You are about ...
-----
Country Name (2 letter code) [AU]:FR
State or ... Name (full name) [Some...]:Var
Locality Name (eg, city) []:Cuers
Organization Name (eg...) [Internet ... Ltd]:InfoLoup
Organizational Unit Name (eg...) []:Blog Loup Virtuel
Common Name (e.g... FQDN ...) []:loupvirtuel.fr
Email Address []:admin@loupvirtuel.fr

Please enter ...
-----
A challenge password []:Rien -> Touche Entrée
An optional company name []:Rien -> Touche Entrée
```

Le certificat CSR sera traité par le CA comme une demande de signature.

Il est recommandé aujourd'hui d'ajouter l'extension d'identification de noms de domaines alternatifs _(subjectAltName)_ à la demande de signature.

Pour cela, créez un fichier d'extension de nom csr.ext :

```bash
[srvdmz@srvdmz:~$] sudo nano csr.ext   
```

et entrez ce contenu répondant à la norme X509v3 :

```markdown
authorityKeyIdentifier=keyid,issuer 
basicConstraints=CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = loupvirtuel.fr
DNS.2 = localhost
DNS.3 = srvdmz.loupvirtuel.fr
IP.1 = 127.0.0.1 
```

Enfin, traitez le CSR et son extension en générant un certificat CRT final signé pour Apache :

```bash
[srvdmz@srvdmz:~$] sudo openssl x509 -req -in loupvirtuel.csr -CA loupvirtuel-ca.pem -CAkey loupvirtuel-ca.key -CAcreateserial  -out loupvirtuel.crt -days 3650 -sha256 -extfile csr.ext    
```

Retour pour le certificat final _(CRT)_ :

```markdown
Certificate request self-signature ok
subject=C = FR, ST = Var, L = Cuers, O = InfoLoup, OU = Blog Loup Virtuel, CN = loupvirtuel.fr, emailAddress = admin@loupvirtuel.fr
Enter pass ... loupvirtuel-ca.key: Votre pass phrase CA 
```

!!! note "Nota"
    C'est en tant que CA _(autorité de certification)_ que vous venez de générer le certificat CRT.

Bilan des clés et certificats créés dans /etc/ssl/ :

* loupvirtuel-ca.key
* loupvirtuel-ca.pem
* loupvirtuel.key
* loupvirtuel.csr
* csr.ext
* loupvirtuel-ca.srl
* loupvirtuel.crt

#### _- Certificat PEM dans Firefox_

Afin que le HTTPS fonctionne sans alerte avec un certificat CA auto-signé, il est nécessaire d'importer celui-ci dans la configuration du navigateur Web Firefox.

Pour cela, accédez aux Paramètres du navigateur Web :  
-> Vie privée et sécurité -> Certificats  
-> Bouton Afficher les certificats...

Une fenêtre Gestionnaire de certificats s'ouvre :  
-> Onglet Autorités -> Bouton Importer...

-> /etc/ssl/ -> Sélectionnez loupvirtuel-ca.pem -> Ouvrir

Une fenêtre Téléchargement du certificat s'ouvre :  
-> Cochez les 2 demandes de confirmation  
-> OK -> OK

Vous devriez à présent trouver dans la liste des autorités gérées par Firefox l'Autorité de Certification de nom Internet-CA.

#### _- Hôte virtuel et test HTTPS_

Enfin, depuis la VM `Debian12-vm1`, testez l'URL :  
`http://192.168.4.2`

<figure markdown>
  ![Capture - Apache : Accueil /var/www/html/index.html](../images/2024/04/srvdmz-apache-accueil-deb12.webp){ width="430" }
  <figcaption>Apache : Accueil /var/www/html/index.html</figcaption>
</figure>

![Image - Rédacteur satisfait](../images/2023/07/redacteur_satisfait.jpg "Image Pixabay - Mohamed Hassan"){ align=left }

&nbsp;  
Voilà pour la création de base d'un  
site Web PHP MySQL. La partie 2  
vous attend pour la création d'un  
blog Dotclear sécurisé HTTPS.

!!! Info "Partie 2 en cours de construction"
