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


Enfin, depuis la VM `Debian12-vm1`, testez l'URL :  
`http://192.168.4.2`

<figure markdown>
  ![Capture - Apache : Accueil /var/www/html/index.html](../images/2024/04/srvdmz-apache-accueil-deb12.webp){ width="430" }
  <figcaption>Apache : Accueil /var/www/html/index.html</figcaption>
</figure>

!!! note "Nota"
    Un mémento futur traitera de l'accès au serveur Web depuis Internet.

![Image - Rédacteur satisfait](../images/2023/07/redacteur_satisfait.jpg "Image Pixabay - Mohamed Hassan"){ align=left }

&nbsp;  
Voilà pour la création de base d'un  
site Web PHP MySQL. La partie 2  
vous attend pour la création d'un  
blog Dotclear sécurisé HTTPS.

!!! Info "Partie 2 en cours de construction"
