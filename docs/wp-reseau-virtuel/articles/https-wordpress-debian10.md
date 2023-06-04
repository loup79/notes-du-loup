---
title: "LAMP HTTPS CMS / Debian 10 : 2/2"
date: "2022-04-20"
categories: 
  - "archives"
---

## Mémento 8.2 - HTTPS avec openssl

### 5 - Mise en place du protocole HTTPS

Ce protocole servira à garantir la sécurité et l’intégrité des informations échangées entre le site loupvirtuel.fr et les navigateurs Web.

HTTPS combinera le HTTP avec une couche de chiffrement SSL/TLS.

Vous utiliserez, pour le chiffrement, l'outil openssl installé avec Apache.

Les clés et certificats dédiés au HTTPS seront créés dans le dossier /etc/ssl/.

Le domaine loupvirtuel.fr étant fictif, impossible de demander à une Autorité de Certification existante de le valider. Il faudra en créer une localement.

#### _5.1 - Clé et certificat PEM du Certificate Auth..._

PEM = Format texte codé en Base64. 
CA = Certificate Authority.

Créez une clé privée avec un MDP pour le CA :

\[srvdmz@srvdmz:~$\] cd /etc/ssl 
\[srvdmz@srvdmz:~$\] sudo openssl genrsa -aes128 \-out loupvirtuel-ca.key 2048 

Retour :

```
[sudo] Mot de passe de srvdmz : 
Generating RSA private key, 2048 bit ... (2 primes)
........+++++
....................+++++
e is 65537 (0x010001)
Enter pass ... loupvirtuel-ca.key: Choisissez un MPD
Verifying - Enter pass ... for loupvirtuel-ca.key:
```

et générez le certificat CA associé à cette clé :

\[srvdmz@srvdmz:~$\] sudo openssl req -x509 -new -nodes -key loupvirtuel-ca.key -sha256 -days 3650 \-out loupvirtuel.pem 

Retour pour le Certificate Authority _(CA)_ :

```
[sudo] Mot de passe de srvdmz : 
Enter pass ... loupvirtuel-ca.key: MDP clé privée CA
You are about ...
-----
Country Name (2 letter code) [AU]:FR
State or Prov... Name (full name) [Some-State]:Paris
Locality Name (eg, city) []:Paris
Organization Name (eg...) [Interne... Ltd]:InfoLoup
Organizational Unit Name (eg...) []:Laboratoire-CA
Common Name (e.g. server FQDN or ...) []:InfoLoup CA
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
Organizational Unit Name (eg...) []:Services-Divers
Common Name (e.g. server FQDN ...) []:loupvirtuel.fr
Email Address []:Votre adresse mail

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
IP.1 = 127.0.0.1

Ce fichier d'extension sera ajoutée au CSR pour la demande de signature.

Enfin, traitez le CSR et son extension en générant un certificat CRT final signé pour Apache :

\[srvdmz@srvdmz:~$\] sudo openssl x509 -req -in loupvirtuel.csr -CA loupvirtuel.pem -CAkey loupvirtuel-ca.key -CAcreateserial  \-out loupvirtuel.crt -days 3650 -sha256 -extfile csr.ext 

Retour pour le certificat final _(CRT)_ :

```
[sudo] Mot de passe de srvdmz : 
Signature ok
subject=C = FR, ST = Deux-Sevres, L = Parthenay, O = InfoLoup, OU = Services-Divers, CN = loupvirtuel.fr, emailAddress = Votre adresse mail
Getting CA Private Key
Enter pass ... loupvirtuel-ca.key:MDP clé privée CA
```

_Nota : C'est en tant que CA (autorité de certification) que vous venez de générer le certificat CRT._

#### _5.3 - Ajout du certificat CA dans Chromium_

Pour que le HTTPS fonctionne, il est nécessaire d'importer le certificat CA dans la configuration du navigateur Web Chromium.

Accédez aux Paramètres du navigateur Web, ensuite :  
\> Confidentialité et sécurité > Sécurité  
\> Gérer les certificats  
\> Onglet Autorités > Bouton Importer

Le gestionnaire de fichiers s'ouvre :  
\> Sélectionnez dans /etc/ssl/ le fichier loupvirtuel.pem  
\> Ouvrir

Une fenêtre Autorité de certification s'ouvre :  
\> Cochez les 2 premiers paramètres de confiance  
\> OK

Vous devriez à présent trouver dans le magasin des autorités gérées par Chromium l'Autorité de Certification de nom InfoLoup.

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

Enfin, testez depuis le navigateur Chromium l'URL :  
https://loupvirtuel.fr

[![Capture - HTTPS : Site loupvirtuel.fr sécurisé](/wp-content/uploads/2019/12/srvdmz_https_loupvirtuel_1-430x208.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/12/srvdmz_https_loupvirtuel_1.jpg)

HTTPS : Site loupvirtuel.fr sécurisé

[![Capture - HTTPS : Détail du certificat pour le site loupvirtuel.fr](/wp-content/uploads/2019/12/srvdmz_https_loupvirtuel_2-430x237.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/12/srvdmz_https_loupvirtuel_2.jpg)

HTTPS : Détail du certificat pour le site loupvirtuel.fr

Les 2 captures ci-dessus montrent le site loupvirtuel.fr validé HTTPS.

Par curiosité, complétez la vérification SSL :

\[srvdmz@srvdmz:~$\] cd /etc/ssl
\[srvdmz@srvdmz:~$\] sudo openssl s\_client \\
             -CAfile loupvirtuel.pem -connect loupvirtuel.fr:443

Le caractère \\ indique d'écrire la Cde sur une seule ligne.

Vous devriez trouver dans le retour et dans l'ordre :  
\- depth =1 > verifiy return :1  
\- depth =0 > verifiy return :1  
\- SSL handshake > Vérification : OK  
\- New, TLSv1.3 > Verify return code: 0 (ok)  
\- Post handshake > Verify return code: 0 (ok)  
\- Post handshake > Verify return code: 0 (ok)

Aucun message d'erreur ne doit apparaitre.

#### _5.5 - Tests HTTPS depuis la VM srvlan, etc..._

La configuration actuelle exige, pour joindre le domaine loupvirtuel.fr depuis les VM srvlan et debian10-vm\*, de modifier leurs fichiers DNS respectifs /etc/hosts.

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
\[srvdmz@srvdmz:~$\] sudo cp loupvirtuel.pem /home/srvdmz/Partages/

et importez celui-ci dans les navigateurs Web des VM :

\- Pour srvlan, procédez comme ci-dessus avec srvdmz mais en pensant à sélectionner le fichier loupvirtuel.pem dans /home/srvlan/Partages/.  

\- Pour debian10-vm\*, procédez comme ceci :  
Accédez aux Préférences du navigateur Web Firefox  
\> Vie privée et sécurité > Certificats  
\> Bouton Afficher les certificats...

Une fenêtre Gestionnaire de certificats s'ouvre :  
\> Onglet Autorités > Bouton Importer...

Le gestionnaire de fichiers s'ouvre :  
\> Ouvrez le dossier /home/clientx2dlinux/Partages/  
\> Sélectionnez le fichier loupvirtuel.pem  
\> Ouvrir

Une fenêtre Téléchargement du certificat s'ouvre :  
\> Cochez les 2 demandes de confirmation  
\> OK > OK

Pour finir, testez pour chacune des VM l'accès à l'URL :  
https://loupvirtuel.fr

### 6 - Installation du CMS WordPress

CMS = Content Management System

#### _6.1 - Téléchargement et extraction du CMS_

\[srvdmz@srvdmz:~$\] cd /var/www
\[srvdmz@srvdmz:~$\] sudo wget https://fr.wordpress.org/latest-fr\_FR.tar.gz

\[srvdmz@srvdmz:~$\] sudo tar zxvf latest-fr\_FR.tar.gz
\[srvdmz@srvdmz:~$\] sudo rm latest-fr\_FR.tar.gz    

Un dossier wordpress a été créé dans /var/www/.

#### _6.2 - Préparation de l'installation_

Créez avec le gestionnaire Adminer une Bdd loupvirtuel d'interclassement utf8\_general\_ci :

[![Capture - Adminer : Création d'une Bdd loupvirtuel](/wp-content/uploads/2019/12/srvdmz_creation_bdd_loupvirtuel-430x181.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/12/srvdmz_creation_bdd_loupvirtuel.jpg)

Adminer : Création d'une Bdd loupvirtuel

et ajoutez les modules PHP suivants :

\[srvdmz@srvdmz:~$\] sudo apt install php7.3-curl php7.3-mbstring php-imagick php7.3-zip php7.3-gd php7.3-xml

Créez un fichier de configuration wp-config.php :

\[srvdmz@srvdmz:~$\] cd /var/www/wordpress
\[srvdmz@srvdmz:~$\] sudo cp wp-config-sample.php wp-config.php 

Editez celui-ci :

\[srvdmz@srvdmz:~$\] sudo nano wp-config.php

Modifiez les valeurs de DB\_\* comme suit :

\*\* Réglages MySQL - Votre hébergeur doit ... informations. \*\*
 \*\* Nom de la base de données de WordPress. \*
 define('DB\_NAME', 'loupvirtuel');

 \*\* Utilisateur de la base de données MySQL. \*
 define('DB\_USER', 'root');

 \*\* Mot de passe de la base de données MySQL. \*
 define('DB\_PASSWORD', 'Votre MDP pour le root de MySQL');

 \*\* Adresse de l’hébergement MySQL. \*
 define('DB\_HOST', 'localhost');

et ajoutez ceci sous la ligne define('DB\_COLLATE' ...); :

\# Mises à jour de Wordpress sans utiliser le FTP
 define('FS\_METHOD', 'direct');

Modifiez le propriétaire et le groupe du dossier WP :

\[srvdmz@srvdmz:~$\]  sudo chown -R www-data:www-data /var/www/wordpress 

Editez à présent l'hôte virtuel loupvirtuel créé ci-dessus :

\[srvdmz@srvdmz:~$\] cd /etc/apache2/sites-available
\[srvdmz@srvdmz:~$\] sudo nano loupvirtuel.conf  

et modifiez la section <VirtualHost \*:443> comme suit :

 <VirtualHost \*:443>

DocumentRoot /var/www/wordpress/
ServerName loupvirtuel.fr

<Directory /var/www/wordpress/>
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

Rechargez la configuration de Apache :

\[srvdmz@srvdmz:~$\] sudo systemctl reload apache2

Vous êtes prêt pour une installation normalement sans incident.

#### _6.3 - Installation et test_

Ouvrez Chromium et lancez l'URL https://loupvirtuel.fr.

Puis, remplissez la page présentée par WP :  
\> Titre du site > Loup Virtuel  
\> Identifiant > Un nom qui servira à administrer WP  
\> Mot de passe > Ce que vous voulez  
\> Confirmation du mot de passe si jugé faible  
\> Votre adresse de messagerie > Votre e-mail  
\> Visibilité ... recherche > Cochez la case ne pas indexer  
\> Bouton Installer Wordpress

Si tout est OK, une page de titre Quel succès ! s'affiche.

Cliquez sur Se connecter pour ouvrir l'administration :

[![Capture - WordPress : Panneau d'administration](/wp-content/uploads/2019/12/srvdmz_wordpress_admin-430x231.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/12/srvdmz_wordpress_admin.jpg)

WordPress : Panneau d'administration

Le menu pour se déconnecter se situe en haut à droite.

Les URL à retenir :  
https://loupvirtuel.fr/wp-admin/ pour administrer le site.  
https://loupvirtuel.fr pour parcourir le site.

Amusez-vous maintenant avec WordPress et créez votre propre page d'accueil.

Exemple de page d'accueil :

[![Capture - Wordpress : Page accueil de loupvirtuel.fr](/wp-content/uploads/2019/12/srvdmz_wordpress_accueil-430x237.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/12/srvdmz_wordpress_accueil.jpg)

WordPress : Page d'accueil de loupvirtuel.fr

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Ouf ! Enfin terminé pour ce grand  
chapitre. Le mémento 9.1  
consacré à l'usage du protocole  
SFTP vous attend.

[Mémento 9.1](/sftp-acces-site-wordpress/)
