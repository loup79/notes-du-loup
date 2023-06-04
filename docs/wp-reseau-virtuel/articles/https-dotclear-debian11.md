---
title: "LAMP HTTPS CMS / Debian 11 : 2/2"
date: "2022-03-28"
categories: 
  - "services-web-php-mysql-https"
---

## Mémento 8.21 - HTTPS avec openssl

### 5 - Mise en place du protocole HTTPS

Ce protocole servira à garantir la sécurité et l’intégrité des informations échangées entre le site loupvirtuel.fr et les navigateurs Web.

HTTPS combinera le HTTP avec une couche de chiffrement SSL/TLS.

Vous utiliserez, pour le chiffrement, l'outil openssl installé avec Apache.

Les clés et certificats dédiés au HTTPS seront créés dans le dossier /etc/ssl/.

Le domaine loupvirtuel.fr étant fictif, impossible de demander à une Autorité de Certification existante de le valider. Il faudra en créer une localement.

#### _5.1 - Clé et certificat PEM du Certificate Auth..._

PEM = Format texte codé en Base64. 
CA = Certificate Authority

Créez une clé privée avec un MDP pour le CA :

\[srvdmz@srvdmz:~$\] cd /etc/ssl 
\[srvdmz@srvdmz:~$\] sudo openssl genrsa -aes128 \-out loupvirtuel-ca.key 2048 

Retour :

```
[sudo] Mot de passe de srvdmz : 
Generating RSA private key, 2048 bit ... (2 primes)
...............................+++++
....................+++++
e is 65537 (0x010001)
Enter pass ... loupvirtuel-ca.key: Choisissez un MPD
Verifying - Enter pass phrase for loupvirtuel-ca.key:
```

et générez le certificat CA associé à cette clé :

\[srvdmz@srvdmz:~$\] sudo openssl req -x509 -new -nodes -key loupvirtuel-ca.key -sha256 -days 3650 \-out loupvirtuel-ca.pem 

Retour pour le Certificate Authority _(CA)_ :

```
[sudo] Mot de passe de srvdmz : 
Enter pass ... loupvirtuel-ca.key: MDP clé privée CA
You are about ...
-----
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:Ain
Locality Name (eg, city) []:Bourg-en-Bresse
Organization Name (eg...) [Interne... Ltd]:InfoLoup
Organizational Unit Name (eg...) []:Item certificats
Common Name (e.g... FQDN ...) []:infoloup.no-ip.org
Email Address []:Rien > Touche Entrée
```

#### _5.2 - Clé et certificats CSR/CRT de Apache_

Créez une clé privée pour le serveur Apache :

\[srvdmz@srvdmz:~$\] sudo openssl  genrsa \-out loupvirtuel.key 2048 

Retour :

```
[sudo] Mot de passe de srvdmz : 
Generating RSA private key, 2048 bit ... (2 primes)
...............+++++
..................+++++
e is 65537 (0x010001)
```

et générez le certificat CSR associé à cette clé :

\[srvdmz@srvdmz:~$\] sudo openssl req -new -key loupvirtuel.key \-out loupvirtuel.csr 

Retour pour le Certificate Signing Request _(CSR)_ :

```
[sudo] Mot de passe de srvdmz : 
You are about ...
-----
Country Name (2 letter code) [AU]:FR
State or ... Name (full name) [Some...]:Deux-Sevres
Locality Name (eg, city) []:Parthenay
Organization Name (eg...) [Interne... Ltd]:InfoLoup
Organizational Unit Name (e...) []:Item Loup Virtuel
Common Name (e.g. server FQDN ...) []:loupvirtuel.fr
Email Address []:loup.virtuel@yahoo.fr

Please enter ...
A challenge password []:Rien > Touche Entrée
An optional company name []:Rien > Touche Entrée
```

Ce certificat sera traité par le CA comme une demande de signature.

Il est recommandé aujourd'hui d'ajouter l'extension d'identification de noms de domaines alternatifs subjectAltName à la demande de signature.

Pour cela, créez un fichier d'extension de nom csr.ext :

\[srvdmz@srvdmz:~$\] sudo nano csr.ext

et entrez ce contenu répondant à la norme X509v3 :

authorityKeyIdentifier=keyid,issuer 
basicConstraints=CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment, dataEncipherment
subjectAltName = @alt\_names

\[alt\_names\]
DNS.1 = loupvirtuel.fr
DNS.2 = localhost
DNS.3 = srvdmz.loupvirtuel.fr
IP.1 = 127.0.0.1

Ce fichier d'extension sera ajouté au CSR pour la demande de signature.

Enfin, traitez le CSR et son extension en générant un certificat CRT final signé pour Apache :

\[srvdmz@srvdmz:~$\] sudo openssl x509 -req -in loupvirtuel.csr -CA loupvirtuel-ca.pem -CAkey loupvirtuel-ca.key -CAcreateserial  \-out loupvirtuel.crt -days 3650 -sha256 -extfile csr.ext 

Retour pour le certificat final _(CRT)_ :

```
[sudo] Mot de passe de srvdmz : 
Signature ok
subject=C = FR, ST = Deux-Sevres, L = Parthenay, O = InfoLoup, OU = Item Loup Virtuel, CN = loupvirtuel.fr, emailAddress = loup.virtuel@yahoo.fr
Getting CA Private Key
Enter pass ... loupvirtuel-ca.key:MDP clé privée CA
```

_Nota : C'est en tant que CA _(autorité de certification)__ _que vous venez de générer le certificat CRT._

#### _5.3 - Importation du certificat CA dans Firefox_

Pour que le HTTPS fonctionne, il est nécessaire d'importer le certificat CA dans la configuration du navigateur Web Firefox.

Accédez aux Paramètres du navigateur Web, ensuite :  
\> Vie privée et sécurité > Certificats  
\> Bouton Afficher les certificats...

Une fenêtre Gestionnaire de certificats s'ouvre :  
\> Onglet Autorités > Bouton Importer...

\> /etc/ssl/ > Fichier loupvirtuel-ca.pem > Ouvrir

Une fenêtre Téléchargement du certificat s'ouvre :  
\> Cochez les 2 demandes de confirmation  
\> OK > OK

Vous devriez à présent trouver dans la liste des autorités gérées par Firefox l'Autorité de Certification de nom InfoLoup.

#### _5.4 - Hôte virtuel du site Web et test HTTPS_

Au préalable, demandez à Apache de prendre en compte son module ssl :

\[srvdmz@srvdmz:~$\] sudo a2enmod ssl

Retour :

```
[sudo] Mot de passe de srvdmz : 
Considering dependency setenvif for ssl:
Module setenvif already enabled
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Enabling module socache_shmcb.
Enabling module ssl.
See /usr/share/doc/apache2/README.Debian.gz on ...
To activate the new configuration, you need to run:
  systemctl restart apache2
```

Créez maintenant un hôte virtuel de nom loupvirtuel :

\[srvdmz@srvdmz:~$\] cd /etc/apache2/sites-available
\[srvdmz@srvdmz:~$\] sudo touch loupvirtuel.conf

Editez celui-ci :

\[srvdmz@srvdmz:~$\] sudo nano loupvirtuel.conf 

et entrez le contenu suivant :

<VirtualHost \*:80>
ServerName loupvirtuel.fr
ServerAlias www.loupvirtuel.fr srvdmz.loupvirtuel.fr
Redirect permanent / https://loupvirtuel.fr/
</VirtualHost>

<VirtualHost \*:443>

DocumentRoot /var/www/html/
ServerName loupvirtuel.fr

<Directory /var/www/html/>
     Options -Indexes +FollowSymlinks +MultiViews
     AllowOverride None
     Require all granted
</Directory>

ErrorLog /var/log/apache2/loupvirtuel.error.log
CustomLog /var/log/apache2/loupvirtuel.access.log combined

SSLEngine On
SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire
SSLCertificateFile /etc/ssl/loupvirtuel.crt
SSLCertificateKeyFile /etc/ssl/loupvirtuel.key

</VirtualHost> 

Demandez ensuite à Apache de traiter cet hôte virtuel :

\[srvdmz@srvdmz:~$\] sudo a2ensite loupvirtuel

et redémarrez celui-ci :

\[srvdmz@srvdmz:~$\] sudo systemctl restart apache2

Testez depuis le navigateur Firefox l'URL :  
https://loupvirtuel.fr

Cliquez ensuite sur le cadenas de la barre d'adresse puis sur la ligne Connexion sécurisée :

[![Capture - HTTPS : Site loupvirtuel.fr sécurisé](/wp-content/uploads/2022/03/https_debian11_1-430x202.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/03/https_debian11_1.webp)

HTTPS : Site loupvirtuel.fr sécurisé

Cliquez enfin sur la ligne Plus d'informations et sur Afficher le certificat :

[![Capture - HTTPS : Détail du certificat pour le site loupvirtuel.fr](/wp-content/uploads/2022/03/https_debian11_2-430x352.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/03/https_debian11_2.webp)

HTTPS : Détail du certificat pour le site loupvirtuel.fr

Les 2 captures ci-dessus montrent le site loupvirtuel.fr validé HTTPS.

Par curiosité, complétez la vérification SSL :

\[srvdmz@srvdmz:~$\] cd /etc/ssl
\[srvdmz@srvdmz:~$\] sudo openssl s\_client \\
             -CAfile loupvirtuel-ca.pem -connect loupvirtuel.fr:443

Le caractère \\ indique d'écrire la Cde sur une seule ligne.

Vous devriez trouver dans le retour et dans l'ordre :  
\- depth =1 > verifiy return :1  
\- depth =0 > verifiy return :1  
\- SSL handshake > Vérification : OK  
\- New, TLSv1.3 > Verify return code: 0 (ok)  
\- Post handshake > Verify return code: 0 (ok)  
\- Post handshake > Verify return code: 0 (ok)

Aucun message d'erreur ne doit apparaître.

#### _5.5 - Tests HTTPS depuis la VM srvlan, etc..._

La configuration actuelle exige, pour joindre le domaine loupvirtuel.fr depuis les VM srvlan et debian11-vm\*, de modifier leurs fichiers DNS respectifs /etc/hosts.

Editez chacun des 3 fichiers DNS et ajoutez cette ligne :

192.168.4.2     loupvirtuel.fr

Exemple pour le fichier hosts de la VM srvlan :

```
127.0.0.1	localhost
127.0.1.1	srvlan
192.168.3.1     srvlan.intra.loupipfire.fr srvlan
192.168.4.2     loupvirtuel.fr
```

Puis, copiez le certificat CA dans le dossier partagé :

\[srvdmz@srvdmz:~$\] cd /etc/ssl
\[srvdmz@srvdmz:~$\] sudo cp loupvirtuel-ca.pem /home/srvdmz/Partage/

et importez celui-ci dans les navigateurs Web des VM en procédant comme avec srvdmz mais en sélectionnant le fichier loupvirtuel-ca.pem situé dans /home/srvlan/Partage/ ou /home/clientx2dlinux/Partage/.

Pour finir, testez pour chacune des VM l'accès à l'URL :  
https://loupvirtuel.fr

### 6 - Installation du CMS Dotclear

CMS = Content Management System

Dotclear est un moteur de blog français performant et facile d'utilisation que j'exploite personnellement pour gérer mes notes techniques diverses.

#### _6.1 - Téléchargement et extraction du CMS_

\[srvdmz@srvdmz:~$\] cd /var/www/html
\[srvdmz@srvdmz:~$\] sudo wget https://download.dotclear.net/latest.tar.gz

\[srvdmz@srvdmz:~$\] sudo tar zxvf latest.tar.gz
\[srvdmz@srvdmz:~$\] sudo rm latest.tar.gz    

Un dossier dotclear a été créé dans /var/www/html.

#### _6.2 - Préparation de l'installation_

Créez avec le gestionnaire Adminer une Bdd loupvirtuel d'interclassement utf8\_general\_ci :

[![Capture - Adminer : Création d'une Bdd loupvirtuel](/wp-content/uploads/2022/03/creation_bdd_loupvirtuel_debian11-430x164.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/03/creation_bdd_loupvirtuel_debian11.webp)

Adminer : Création d'une Bdd loupvirtuel

et ajoutez les modules PHP suivants :

\[srvdmz@srvdmz:~$\] sudo apt install php7.4-mbstring php7.4-gd php7.4-xml

Modifiez le propriétaire et le groupe du dossier dotclear :

\[srvdmz@srvdmz:~$\] sudo chown -R www-data:www-data /var/www/html/dotclear 

Editez ensuite l'hôte virtuel loupvirtuel créé ci-dessus :

\[srvdmz@srvdmz:~$\] cd /etc/apache2/sites-available
\[srvdmz@srvdmz:~$\] sudo nano loupvirtuel.conf  

et modifiez sa section <VirtualHost \*:443> comme suit :

 <VirtualHost \*:443>

DocumentRoot /var/www/html/dotclear/
ServerName loupvirtuel.fr

<Directory /var/www/html/dotclear/>
     Options -Indexes +FollowSymlinks +MultiViews
     AllowOverride ALL
     Require all granted
</Directory>

ErrorLog /var/log/apache2/loupvirtuel.error.log
CustomLog /var/log/apache2/loupvirtuel.access.log combined

SSLEngine On
SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire
SSLCertificateFile /etc/ssl/loupvirtuel.crt
SSLCertificateKeyFile /etc/ssl/loupvirtuel.key

</VirtualHost>  

Rechargez enfin la configuration de Apache :

\[srvdmz@srvdmz:~$\] sudo systemctl reload apache2

Vous êtes maintenant prêt pour effectuer une installation de Dotclear sans incident.

#### _6.3 - Installation et test_

Ouvrez Firefox et lancez l'URL https://loupvirtuel.fr.

Une page Assistant d'installation de Dotclear s'ouvre :  
\> Type de base de données : MySQLi  
\> Nom d'hôte de la base de données : localhost  
\> Nom de la base de données : loupvirtuel  
\> Nom d'utilisateur de la base de données : root  
\> MDP de la base de données : Votre MDP root Mysql  
\> Préfixe des tables de la base de données : dc\_  
\> Email principal : Votre adresse Email personnelle  
  
\> Bouton Continuer

Une page Installation de Dotclear s'ouvre :  
\- Informations utilisateur  
\> Prénom : Votre prénom  
\> Nom : Votre nom  
\> Email : Votre adresse Email personnelle

\- Identifiant et mot de passe  
\> Nom d'utilisateur : Ce que vous voulez  
\> Nouveau mot de passe : Ce que vous voulez  
\> Confirmer le mot de passe :  
  
\> Bouton Enregistrer

Si tout est OK, une page titrée C'est terminé ! s'affiche :  
\> Bouton Gérer votre blog pour l'accès à l'administration

[![Capture - Dotclear : Panneau d'administration](/wp-content/uploads/2022/03/dotclear-administration-430x238.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/03/dotclear-administration.webp)

Dotclear : Panneau d'administration

Le menu pour se déconnecter se situe en haut à droite.

Les URL à retenir :  
https://loupvirtuel.fr/admin/ pour administrer le site.  
https://loupvirtuel.fr pour parcourir le site.

Amusez-vous maintenant avec Dotclear et créez votre propre page d'accueil.

Exemple de page d'accueil :

[![Capture - Dotclear : Page accueil de loupvirtuel.fr](/wp-content/uploads/2022/03/dotclear-accueil-430x188.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/03/dotclear-accueil.webp)

Dotclear : Page d'accueil de loupvirtuel.fr

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà ! Terminé pour ce grand  
chapitre. Le mémento 9.11 vous  
attend pour découvrir l'usage du  
protocole SFTP.

[Mémento 9.11](https://familleleloup.no-ip.org/sftp-dotclear-debian11/)
