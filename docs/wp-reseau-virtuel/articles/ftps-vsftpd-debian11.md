---
title: "FTPS - VsFTPd / Debian 11"
date: "2022-05-02"
categories: 
  - "services-sftp-ftps"
---

## Mémento 9.21 - Serveur VsFTPd

Le serveur VsFTPd sera installé sur la VM srvlan.

Le FTP est un protocole de communication dédié au transfert de fichiers sur un réseau TCP/IP.

Les utilisateurs des VM debian11-vm\* _(clients)_ utiliseront la VM srvlan _(serveur)_ pour déposer ou s'échanger des fichiers.

Selon le paramétrage FTP, un utilisateur pourra ou non Déposer, Télécharger, Lire, Modifier ou Supprimer des dossiers/fichiers.

### 1 - Préambule

Outils utilisés :  
\- Serveur VsFTPd _(Very secure FTP daemon)_.  
\- Console client lftp à la place de ftp.

Configurations proposées :  
\- Usage du FTPS _(FTP via SSL)_.  
\- Usage du mode passif.  

#### _1.1 - Rôle d'un serveur FTP_

Il permet le transfert de dossiers/fichiers avec un client distant, par Internet ou par le réseau informatique local. Le transfert, selon un modèle client/serveur, peut se faire dans les 2 sens.

Le serveur FTP d'un hébergeur permet par exemple à un client distant de déposer ou télécharger les fichiers de son site web.

En général, deux modes de connexion sont utilisés :  
Le mode anonyme qui autorise l'accès au dossier public du serveur sans fournir de nom d'utilisateur ni de MDP. Les fichiers de ce dossier sont souvent en lecteur seule.  
  
Le mode authentifié qui permet à un utilisateur local ou virtuel, après contrôle de son nom et de son MDP, d'accéder en lecture/écriture à ses propres fichiers.

### 2 - Installation

#### _2.1 - Serveur vsftpd_

Au préalable, mettez à jour la distribution Debian 11 :

\[srvlan@srvlan:~$\] sudo apt update
\[srvlan@srvlan:~$\] sudo apt upgrade
\[srvlan@srvlan:~$\] sudo reboot

et installez vsftpd pour son côté léger et sécurisé :

\[srvlan@srvlan:~$\] sudo apt install vsftpd

Vérifiez le bon démarrage de celui-ci :

\[srvlan@srvlan:~$\] sudo systemctl status vsftpd

Retour :

```
● vsftpd.service - vsftpd FTP server
   Loaded: loaded (/lib/systemd/system/vsftpd...
   Active: active (running) since Mon 2022-... ago
  Process: 1575 ExecStartPre=/bin/mkdir -p /var...
 Main PID: 1576 (vsftpd)
    Tasks: 1 (limit: 1116)
   Memory: 948.0K
      CPU: 33ms
   CGroup: /system.slice/vsftpd.service
             └─1576 /usr/sbin/vsftpd /etc/vsftd...

mai 02 ... srvlan systemd[1]: Starting vsftpd ...
mai 02 ... srvlan systemd[1]: Started vsftpd ...
```

Un utilisateur de nom ftp a été créé lors de l'installation.  
Un dossier personnel /srv/ftp/ lui a été associé.  
Il s'agit du dossier public FTP par défaut du serveur.

Pour les tests, copiez ces 2 fichiers dans ce dossier :

\[srvlan@srvlan:~$\] cd /usr/share/doc/vsftpd
\[srvlan@srvlan:~$\] sudo cp README /srv/ftp/README
\[srvlan@srvlan:~$\] sudo cp TODO /srv/ftp/TODO

#### _2.2 - Sécurisation des échanges FTP via SSL_

De base le serveur vsftpd ne chiffre rien mais il peut utiliser le protocole FTPS _(FTP via SSL)_ pour chiffrer ses communications avec les clients.

Pour information, différences entre le SFTP et le FTPS :  
\- SFTP plus récent utilise par défaut le port 22 pour traiter l’authentification, les Cdes FTP et le transfert de données.

La gestion d'un pare-feu est facilitée, un seul port.

\- FTPS utilise par défaut le port 21 pour traiter l'authentification et les Cdes FTP ainsi que le port 20 _(en mode actif)_ ou une tranche de ports _(en mode passif)_ pour le transfert de données.

Un pare-feu sera donc plus compliqué à gérer.

Pour le FTPS, installez si absent le paquet openssl :

\[srvlan@srvlan:~$\] sudo apt install openssl

Puis connectez-vous en tant que root et créez un certificat CA _(Certificate Authority)_ auto-signé :

\[srvlan@srvlan:~$\] su root
\[root@srvlan:~#\] cd /etc/ssl/private
\[root@srvlan:~#\] openssl req -x509 -nodes -newkey rsa:2048 -keyout vsftpd.pem -out vsftpd.pem -days 3650

en répondant aux questions posées comme suit :

```
Country Name (2 letter code) [AU]:FR
State or Prov... Name (full name) [Some-State]:Paris
Locality Name (eg, city) []:Paris
Organization Name (e...) [Interne...]:Intra-InfoLoup            
Organizational Uni... (eg...) []:Intra-Labo-FTPS
Common Name (... server FQDN or YOUR name) []:srvlan
Email Address []:Rien > Touche Entrée
```

Editez ensuite le fichier de configuration vsftpd.conf :

\[root@srvlan:~#\] exit

\[srvlan@srvlan:~$\] sudo nano /etc/vsftpd.conf

et ajustez vers la ligne 147 les paramètres SSL :

\# This option specifies the location of the RSA certificate ...
# encrypted connections.
rsa\_cert\_file=/etc/ssl/private/vsftpd.pem
rsa\_private\_key\_file=/etc/ssl/private/vsftpd.pem
ssl\_enable=YES

ssl\_ciphers=HIGH
ssl\_tlsv1=YES
ssl\_sslv2=NO
ssl\_sslv3=NO
force\_local\_data\_ssl=YES
force\_local\_logins\_ssl=YES
allow\_anon\_ssl=NO

Le contenu des 7 dernières lignes ajoutées signifie :  
\- Niveau de chiffrement élevé imposé.  
\- Protocole ssl\_tlsv1 autorisé.  
\- Protocole ssl\_sslv2 refusé car moins sécurisé.  
\- Protocole ssl\_sslv3 refusé car moins sécurisé.  
\- Login non anonyme forcé SSL pour l'envoi des Data.  
\- Login non anonyme forcé SSL pour l'envoi du MDP.  
\- Login anonyme non forcé SSL.

#### _2.3 - Activation du mode FTP passif_

Le serveur fonctionne par défaut en mode actif. Il vaut mieux, pour éventuellement éviter que le pare-feu d'un client distant situé sur Internet ne bloque la liaison FTPS, passer celui-ci en mode passif.

Ouvrez le [synoptique du mémento](/wp-content/uploads/2022/05/ftp-ftps-serveur-vsftpd-memento-9.21.webp) pour découvrir la différence entre les 2 modes.

Activez l'usage du mode passif en fin de vsftpd.conf :

\# Activation du mode passif
pasv\_enable=YES
pasv\_min\_port=12500         \# La tranche de ports  aléatoires 
pasv\_max\_port=12550        \# doit-être > à 1024

et redémarrez le serveur FTP :

\[srvlan@srvlan:~$\] sudo systemctl restart vsftpd
\[srvlan@srvlan:~$\] sudo systemctl status vsftpd

### 3 - Connexions anonymes

#### _3.1 - Configuration de base_

Par défaut, vsftpd refuse les connexions anonymes.  
Pour changer cela, éditez son fichier de configuration :

\[srvlan@srvlan:~$\] sudo nano /etc/vsftpd.conf

et réglez les valeurs de ces 3 paramètres comme suit :

anonymous\_enable=YES                    \# Accès anonyme autorisé
anon\_upload\_enable=NO                         \# Dépôt fichiers interdit
anon\_mkdir\_write\_enable=NO        \# Création dossiers interdite

N'oubliez pas de retirer si besoin le # de début de ligne.

Redémarrez le serveur FTP :

\[srvlan@srvlan:~$\] sudo systemctl restart vsftpd

#### _3.2 - Connexion distante depuis une console_

Installez le client FTP de nom lftp sur debian11-vm2 :

\[client-linux@debian11-vm2:~$\] sudo apt install lftp

Ce programme permet d'envoyer des lignes de Cdes vers un serveur FTP/FTPS/SFTP.

Créez un fichier caché .lftprc qui sera utilisé par lftp :

\[client-linux@debian11-vm2:~$\] cd /home/client-linux
\[client-linux@debian11-vm2:~$\] nano .lftprc

et entrez la configuration suivante :

\# Configuration traitée au démarrage du client lftp
set ftp:passive-mode true
set ftp:ssl-auth TLS
set ftp:ssl-force true
set ftp:ssl-protect-data true
set ftp:ssl-protect-list true
set ssl:verify-certificate no

Enfin, connectez-vous sur le serveur vsftpd de srvlan :

\[client-linux@debian11-vm2:~$\] lftp srvlan
lftp srvlan:~> ls

La Cde ls renvoie le contenu du dossier public /srv/ftp/ :

```
lftp srvlan:~> ls
-rw-r--r-- 1 0      0  1361 May 02 13:54 README
-rw-r--r-- 1 0      0  1833 May 02 13:54 TODO
lftp srvlan:/> quit
```

Utilisez la Cde quit pour fermer la connexion distante.

Testez ensuite un téléchargement FTP :

\[client-linux@debian11-vm2:~$\] cd Documents
\[client-linux@debian11-vm2:~$\] lftp srvlan
lftp srvlan:~> get TODO

Retour :

```
lftp srvlan:~> get TODO
1833 octets transférés                        
lftp srvlan:/> quit
```

Créez enfin un fichier de test test-ftp-vm2 comme suit :

\[client-linux@debian11-vm2:~$\] touch test-ftp-vm2
\[client-linux@debian11-vm2:~$\] echo "Test Upload FTP" > test-ftp-vm2

et testez un envoi FTP dans le dossier public de srvlan :

\[client-linux@debian11-vm2:~$\] lftp srvlan
lftp srvlan:~> put test-ftp-vm2

Retour :

```
lftp srvlan:~> put test-ftp-vm2
put: L'accès a échoué : 550 Permission denied. (te...
lftp srvlan:/> quit

```

Echec normal puisque nous avons plus haut interdit le dépôt anonyme de fichiers.

#### _3.3 - Connexion distante depuis FileZilla_

Installez sur debian11-vm2 le client FTP/SFTP FileZilla :

\[client-linux@debian11-vm2:~$\] sudo apt install filezilla

FileZilla fonctionne en mode passif par défaut.

Ouvrez ensuite l'application graphique :  
\- Menu Applications > Internet > Icône FileZilla

Cliquez sur l'icône située la plus à gauche de la barre des icônes afin d'ouvrir le Gestionnaire de Sites et cliquez sur le bouton Nouveau site.

Créez un site de nom srvlan-anonymous comme suit :  
\- Onglet Général

[![Capture - FileZilla : Réglages généraux FTP Anonymous](/wp-content/uploads/2022/05/ftps-deb11-anonymous-1-430x237.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/05/ftps-deb11-anonymous-1.webp)

FileZilla : Réglages généraux FTP Anonymous

\- Onglet Avancé

[![Capture - FileZilla : Réglages avancés FTP Anonymous](/wp-content/uploads/2022/05/ftps-deb11-anonymous-2-430x236.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/05/ftps-deb11-anonymous-2.webp)

FileZilla : Réglages avancés FTP Anonymous

Une fenêtre Se souvenir des mots de passe ? s'ouvre :  
\> Cochez Sauvegarder les mots de passe > Valider

Le site srvlan-anonymous est à présent créé. Cliquez sur la flèche située à droite de l'icône du Gestionnaire de Sites et sélectionnez celui-ci.

Une fenêtre Connexion FTP non sécurisée s'ouvre :  
\> Cochez Toujours autoriser FTP non sécurisé pour ...  
\> Valider

La connexion s'établit et la zone Site distant doit montrer le contenu du dossier public de srvlan.

[![Capture - FileZilla : Connexion FTP Anonymous établie](/wp-content/uploads/2022/05/ftps-deb11-anonymous-3-430x247.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/05/ftps-deb11-anonymous-3.webp)

FileZilla : Connexion FTP Anonymous établie

Pour Afficher/Editer les fichiers texte, procédez ainsi :  
\> Menu Edition de FileZilla > Paramètres...  
\> Edition des fichiers  
\> Cochez Utiliser l'éditeur person... > /usr/bin/mousepad  
\> Cochez Toujours utiliser l'éditeur par défaut  
\> Valider

Vérifiez à présent que vous pouvez :  
\- En zone Site local  
Télécharger un fichier mais non en Envoyer _(Upload)_.

\- En zone Site distant  
Afficher/Editer un fichier mais non le Renommer ni le Supprimer.

Cliquez sur l'icône serveur/croix-rouge pour fermer la connexion.

#### _3.4 - Configuration ouvrant l'Upload de fichiers_

Choix déconseillé pour raison de sécurité, un anonyme pouvant déposer un fichier infecté.

Créez et affectez un dossier uploads à l'utilisateur ftp :

\[srvlan@srvlan:~$\] sudo mkdir /srv/ftp/uploads
\[srvlan@srvlan:~$\] sudo chown ftp:ftp /srv/ftp/uploads

Donnez à l'utilisateur ftp le droit d'écriture sur celui-ci :

\[srvlan@srvlan:~$\] sudo chmod 755 /srv/ftp/uploads

et retirez lui ce droit sur le dossier racine /srv/ftp/ :

\[srvlan@srvlan:~$\] sudo chmod 555 /srv/ftp

Editez ensuite le fichier de configuration vsftpd.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/vsftpd.conf

puis ajustez les paramètres ci-dessous comme suit :

write\_enable=YES                   \# Ecriture autorisée
anon\_upload\_enable=YES                  \# Dépôt fichiers autorisé
anon\_mkdir\_write\_enable=YES \# Création dossiers autorisée

et ajoutez les 2 lignes suivantes en fin de fichier :

\# Permissions affectées au contenu du dossier uploads 
anon\_umask=022               \# 644 fichiers et 755 dossiers

Pour finir, redémarrez le serveur FTP :

\[srvlan@srvlan:~$\] sudo systemctl restart vsftpd

#### _3.5 - Test de bon fonctionnement de l'Upload_

Accédez au dossier suivant de la VM debian11-vm2 :

client-linux@debian11-vm2:~$\] cd /home/client-linux
client-linux@debian11-vm2:~$\] cd Documents

et testez le dépôt _(upload)_ suivant avec la Cde lftp :

```
client-linux@debian11-vm2:~/Documents$ lftp srvlan
lftp srvlan:~> ls
-rw-r--r-- 1 0    0      1361 May 02 13:54 README
-rw-r--r-- 1 0    0      1833 May 02 13:54 TODO
drwxr-xr-x    2 117  125    4096 May 07 ... uploads
lftp srvlan:/> cd uploads
lftp srvlan:/uploads> put test-ftp-vm2
16 octets transférés
lftp srvlan:/uploads> ls
-rw-r--r-- 1 117  125    16 May... test-ftp-vm2
lftp srvlan:/uploads> quit
client-linux@debian11-vm2:~/Documents$
```

Testez ensuite le dépôt avec l'outil graphique FileZilla.

Vérifiez l'impossibilité de Supprimer, Renommer et Modifier un fichier/dossier déposé dans le dossier uploads.

Vérifiez également l'impossibilité maintenant de réaliser un envoi FTP en dehors du dossier uploads _(droits du dossier racine /srv/ftp/ fixés ci-dessus à 555)_.

### 4 - Connexions authentifiées d'utilisateurs locaux

#### _4.1 - Accès FTP limité à un dossier commun_

C'est actuellement l'utilisateur nobody de Linux qui permet au démon vsftpd d'exécuter ses processus enfants. L'inconvénient est que d'autres démons peuvent également utiliser nobody.

Il est donc, par sécurité, conseillé de créer un utilisateur qui aura des permissions dédiées au démon vsftpd et de le déclarer dans vsftpd.conf à l'aide du paramètre nopriv\_user.

Pour cela, vous allez créer un utilisateur local de nom userftps sans MDP associé, ce qui par sécurité, interdira toute tentative de connexion avec ce nom sur srvlan.

Un dossier /home/ftp/ affecté à cet utilisateur et au groupe grftps servira de racine aux utilisateurs locaux et plus tard aux utilisateurs virtuels.

Créez le dossier/groupe/utilisateur et les permissions :

\[srvlan@srvlan:~$\] sudo mkdir -p /home/ftp
\[srvlan@srvlan:~$\] sudo groupadd grftps

\[srvlan@srvlan:~$\] sudo useradd -g grftps -d /home/ftp userftps

\[srvlan@srvlan:~$\] sudo chown userftps:grftps /home/ftp
\[srvlan@srvlan:~$\] sudo chmod 555 /home/ftp

Créez ensuite les utilisateurs locaux clientvm\* suivants :

\[srvlan@srvlan:~$\] sudo adduser clientvm1
\[srvlan@srvlan:~$\] sudo adduser clientvm2

Un MDP UNIX sera demandé pour chacun d'eux. Vous pouvez utiliser ceux des VM debian11-vm\* mais ce n'est pas obligatoire. Vous êtes également libre de répondre ou non aux demandes d'information qui apparaîtront.

Ajoutez les 2 utilisateurs au groupe grftps :

\[srvlan@srvlan:~$\] sudo adduser clientvm1 grftps
\[srvlan@srvlan:~$\] sudo adduser clientvm2 grftps

et créez pour ceux-ci le dossier commun\_locaux :

\[srvlan@srvlan:~$\] cd /home/ftp
\[srvlan@srvlan:~$\] sudo mkdir commun\_locaux

\[srvlan@srvlan:~$\] sudo chown userftps:grftps commun\_locaux

\[srvlan@srvlan:~$\] sudo chmod 775 commun\_locaux
\[srvlan@srvlan:~$\] sudo chmod +t commun\_locaux \# sticky bit

L'activation du stiky bit interdira aux utilisateurs locaux de Supprimer/Modifier les fichiers situés dans le dossier commun\_locaux dont ils ne sont pas propriétaires.

Vérifiez toutes les créations dans /home/ ainsi que le contenu de ces 3 fichiers :

\[srvlan@srvlan:~$\] sudo cat /etc/group
\[srvlan@srvlan:~$\] sudo cat /etc/passwd
\[srvlan@srvlan:~$\] sudo cat /etc/shadow

Editez à présent le fichier de configuration vsftpd.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/vsftpd.conf

puis ajustez les paramètres suivants comme affichés :

local\_enable=YES                          \# Accès utilisateur local=YES
write\_enable=YES                       \# Ecriture utilisateur local=YES
local\_umask=022       \# 644 fichiers reçus, 755 dossiers reçus
nopriv\_user=userftps        \# Utilisateur non privilégié de vsftpd
ftpd\_banner=Serveur FTP de ...                    \# Accueil de vsftpd
chroot\_local\_user=YES      \# Utilisateur local limité à sa racine
secure\_chroot\_dir=/home/ftp  \# Racine utilisateur de userftps

\- ftpd\_banner : Permet de cacher la version de vsftpd.

et ajoutez ces 3 lignes en fin de fichier :

\# Gestion des utilisateurs locaux
local\_root=/home/ftp/commun\_locaux      \# Racine utilisateurs
allow\_writeable\_chroot=YES     \# Ecriture dans racine autorisée

Redémarrez le serveur FTP :

\[srvlan@srvlan:~$\] sudo systemctl restart vsftpd

Créez la note suivante dans le dossier commun\_locaux :

\[srvlan@srvlan:~$\] cd /home/ftp/commun\_locaux
\[srvlan@srvlan:~$\] sudo nano note-ftps-utilisateurs-locaux

et entrez ce contenu précisant le rôle du dossier :

Ici, dossier FTP dédié aux utilisateurs locaux.

#### _4.2 - Connexion sur le dossier commun_

Créez sur le client FileZilla de debian11-vm2 un site de nom srvlan-commun.

Paramétrez celui-ci comme montré ci-dessous :  
\- Onglet Général

[![Capture - FileZilla : Création du site FTPS srvlan-commun](/wp-content/uploads/2022/05/ftps-deb11-commun-1-430x237.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/05/ftps-deb11-commun-1.webp)

FileZilla : Création du site FTPS srvlan-commun

\- Onglet Avancé  
\> Dossier local par défaut > Parcourir...  
\> Sélectionnez /home/client-linux/Documents  
\> Valider

Lancez ensuite une connexion FTPS sur le site.

1 ) Traitez le certificat SSL présenté comme suit :

[![Capture - FileZilla : Certificat SSL émis par le serveur vsftpd](/wp-content/uploads/2022/05/ftps-deb11-cert-ssl-430x416.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/05/ftps-deb11-cert-ssl.webp)

FileZilla : Certificat SSL émis par le serveur vsftpd

2 ) La connexion est établie :

[![Capture - FileZilla : Connexion sécurisée FTPS établie](/wp-content/uploads/2022/05/ftps-deb11-commun-2-430x246.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/05/ftps-deb11-commun-2.webp)

FileZilla : Connexion sécurisée FTPS établie

Le transfert FTP doit fonctionner dans les 2 sens.

Mais vous ne devez pas pouvoir, zone Site distant, Supprimer, Renommer et Modifier les fichiers déposés sur srvlan par d'autres utilisateurs, cas du fichier note-ftps-utilisateurs-locaux.

Vous devez en revanche pouvoir lire leur contenu.

Pour finir, créez sur le client FileZilla de debian11-vm1 le site srvlan-commun avec comme utilisateur local clientvm1 et contrôlez après connexion que vous retrouvez bien les mêmes permissions que ci-dessus.

#### _4.3 - Accès FTP à tous les dossiers_

Choix déconseillé, l'utilisateur du FTP pouvant accéder à des fichiers sensibles.

La configuration actuelle enferme les utilisateurs locaux tels clientvm\* dans le dossier racine commun\_locaux, ceux-ci ne peuvent donc pas accéder au dossier parent donc à l'arborescence complète de srvlan.

Ce comportement sécurisé vient de ces paramètres :

chroot\_local\_user=YES       \# Utilisateur local limité à sa racine 
local\_root=/home/ftp/commun\_locaux                         \# Racine

Mais vous pouvez avoir besoin qu'un utilisateur local accède à toute l'arborescence de srvlan ceci avec des permissions différentes de celles de son dossier personnel et du dossier racine commun\_locaux.

Editez pour cela le fichier vsftpd.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/vsftpd.conf

puis décommentez les 2 lignes suivantes :

chroot\_list\_enable=YES  
chroot\_list\_file=/etc/vsftpd.chroot\_list

et redémarrez le serveur FTP :

\[srvlan@srvlan:~$\] sudo systemctl restart vsftpd

Créez maintenant un utilisateur local de nom indiscret :

\[srvlan@srvlan:~$\] sudo adduser indiscret

Pour simplifier, donnez lui le MDP indiscret.

Affectez ensuite celui-ci au groupe grftps :

\[srvlan@srvlan:~$\] sudo adduser indiscret grftps

Pour finir, créez le fichier vsftpd.chroot\_list :

\[srvlan@srvlan:~$\] sudo nano /etc/vsftpd.chroot\_list

et entrez l'utilisateur indiscret en début de celui-ci :

indiscret

Si d'autres noms, placez ceux-ci les uns sous les autres.

Créez sur le FileZilla de debian11-vm1 le site srvlan-complet et attribuez lui ces paramètres :

\- Onglet Général  
\> Hôte > srvlan  
\> Chiffrement > Connexion FTP explicite sur TLS si dis...  
\> Type d'authentification > Normale  
\> Identifiant > indiscret  
\> Mot de passe > indiscret

\- Onglet Avancé  
\> Dossier local par défaut > Parcourir...  
\> Sélectionnez /home/client-linux/Documents  
\> Dossier distant par défaut  
\> Entrez /home/indiscret  
\> Valider

Lancez une connexion sur le site srvlan-complet :

[![Capture - FileZilla : Accès complet aux dossiers/fichiers de srvlan](/wp-content/uploads/2022/05/ftps-deb11-acces-complet-430x238.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/05/ftps-deb11-acces-complet.webp)

FileZilla : Accès complet aux dossiers/fichiers de srvlan

et vérifiez que rien, en dehors de /home/indiscret/ et /home/ftp/commun\_locaux/, ne peut être supprimé ou modifié mais le droit de lire et télécharger tous les fichiers de srvlan représente un risque de sécurité.

### 5 - Connexions authentifiées d'utilisateurs virtuels

Choix conseillé pour raison de sécurité.

Les utilisateurs virtuels non déclarés avec la Cde adduser seront comme l'utilisateur local userftps inconnus du système mais verront leurs demandes de connexion mappées vers celui-ci, chacun dans leur dossier respectif.

Il s’agit là d’une protection plus efficace que la création d'utilisateurs locaux.

La configuration s’opère en 4 étapes :  
\- Création d’une Bdd Berkeley utilisateurs virtuels  
\- Liaison de la Bdd avec le module PAM de vsftpd  
\- Création des permissions et dossiers des utilisateurs  
\- Configuration de vsftpd et test de connexion

#### _5.1 - Bdd Berkeley des utilisateurs virtuels_

Installez au préalable le paquet db5.3-util :

\[srvlan@srvlan:~$\] sudo apt install db5.3-util

puis créez ensuite le dossier et le fichier suivants :

\[srvlan@srvlan:~$\] sudo mkdir /etc/vsftpd
\[srvlan@srvlan:~$\] sudo touch /etc/vsftpd/virtuels.txt 

Editez le fichier créé virtuels.txt :

\[srvlan@srvlan:~$\] sudo nano /etc/vsftpd/virtuels.txt

et entrez, l'un sous l'autre, ces utilisateurs et MDP :

clientvm1-virtuel                    \# Utilisateur inconnu de srvlan
MDP de clientvm1-virtuel \# ≠ ou non de celui de debian11-vm1
clientvm2-virtuel \# Utilisateur inconnu de srvlan
MDP de clientvm2-virtuel  \# ≠ ou non de celui de debian11-vm2

Terminez la liste par un retour de chariot.

Créez enfin la Bdd virtuels.db pour le module PAM :

\[srvlan@srvlan:~$\] cd /etc/vsftpd
\[srvlan@srvlan:~$\] sudo db5.3\_load -T -t hash -f virtuels.txt virtuels.db 

et autorisez la lecture des 2 fichiers à root uniquement :

\[srvlan@srvlan:~$\] sudo chmod 600 /etc/vsftpd/virtuels.db
\[srvlan@srvlan:~$\] sudo chmod 600 /etc/vsftpd/virtuels.txt 

#### _5.2 - Liaison Bdd avec le module PAM vsftpd_

Editez le fichier de configuration PAM suivant :

\[srvlan@srvlan:~$\] sudo nano /etc/pam.d/vsftpd

et ajoutez, sous \# Note : vsftpd..., ces 2 instructions :

auth sufficient /lib/x86\_64-linux-gnu/security/pam\_userdb.so db=/etc/vsftpd/virtuels

account sufficient /lib/x86\_64-linux-gnu/security/pam\_userdb.so db=/etc/vsftpd/virtuels

Mettre i386-linux-gnu à la place de x86\_64-linux-gnu pour un système 32 bits.

Le module pam\_userdb _(Pluggable Authentication Modules)_ vérifiera les authentifications utilisateurs virtuels/MDP avec les valeurs stockées dans la Bdd Berkeley.

#### _5.3 - Permissions et dossiers des utilisateurs_

Créez le dossier qui contiendra les permissions :

\[srvlan@srvlan:~$\] cd /etc/vsftpd
\[srvlan@srvlan:~$\] sudo mkdir vsftpd\_user\_conf

Créez le fichier des permissions pour clientvm1-virtuel :

\[srvlan@srvlan:~$\] cd vsftpd\_user\_conf
\[srvlan@srvlan:~$\] sudo nano clientvm1-virtuel

et entrez pour cet utilisateur les permissions suivantes :

\# Configuration vsftpd pour l'utilisateur "clientvm1-virtuel"
# Paramétrage pour les mêmes droits qu'un utilisateur local

# Activation de l'utilisateur comme utilisateur virtuel
guest\_enable=YES

# Ecriture autorisée
write\_enable=YES

# Modification et suppression autorisées
anon\_other\_write\_enable=YES

# Download autorisé même si un fichier
# n'est pas en lecture pour tous
anon\_world\_readable\_only=NO

# Dossier dans lequel sera enfermé l'utilisateur
local\_root=/home/ftp/clientvm1-virtuel 

Créez maintenant son dossier personnel :

\[srvlan@... ~$\] cd /home/ftp
\[srvlan@... ~$\] sudo mkdir clientvm1-virtuel

\[srvlan@... ~$\] sudo chown userftps:grftps clientvm1-virtuel
\[srvlan@... ~$\] sudo chmod 755 clientvm1-virtuel 

Créez une note qui permettra d'identifier le dossier :

\[srvlan@srvlan:~$\] cd /home/ftp/clientvm1-virtuel
\[srvlan@srvlan:~$\] sudo nano note-clientvm1-virtuel 

et entrez le texte suivant :

Ici, dossier FTP personnel de "clientvm1-virtuel".

Effectuez les mêmes opérations que ci-dessus pour l'utilisateur virtuel clientvm2-virtuel.

#### _5.4 - Réglage de vsftpd et test de connexion_

Editez le fichier de configuration de serveur FTP :

\[srvlan@srvlan:~$\] sudo nano /etc/vsftpd.conf

et ajoutez les lignes suivantes à la fin de celui-ci :

\# Gestion des utilisateurs virtuels
guest\_username=userftps     \# Valide le mappage vers userftps
user\_config\_dir=/etc/vsftpd/vsftpd\_user\_conf \# Configurations
virtual\_use\_local\_privs=YES \# Mêmes droits utilisateurs locaux

Redémarrez le serveur :

\[srvlan@srvlan:~$\] sudo systemctl restart vsftpd

Créez depuis FileZilla les sites srvlan-clientvm\*-virtuel sur les VM debian11-vm\* comme suit :

\- Onglet Général  
\> Hôte > srvlan  
\> Chiffrement > Connexion FTP explicite sur TLS si dis...  
\> Type d'authentification > Normale  
\> Identifiant > clientvm1-virtuel ou clientvm2-virtuel  
\> Mot de passe > MDP de clientvm1-virtuel ou clientv...

Pour les MDP, voir le [§ 5.1](/ftps-vsftpd-debian11/#51_-_Bdd_Berkeley_des_utilisateurs_virtuels).

\- Onglet Avancé  
\> Dossier local par défaut > Parcourir...  
\> Sélectionnez /home/client-linux/Documents  
\> Valider

Vérifiez après connexion que chaque utilisateur virtuel :  
\- Peut Transférer des fichiers dans les 2 sens.  
\- Peut Supprimer/Modifier ses dépôts sur srvlan.  
\- Reste bien enfermé dans son dossier personnel.

Vérifiez que chaque dossier/fichier déposé sur srvlan est bien mappé sur l'utilisateur userftps :

\[srvlan@srvlan:~$\] ls -l /home/ftp/clientvm1-virtuel
\[srvlan@srvlan:~$\] ls -l /home/ftp/clientvm2-virtuel

### 6 - Configuration préférentielle

En l'état, le serveur FTP accepte les connexions d'utilisateurs anonymes, locaux et virtuels.

Vous l'avez compris, la connexion authentifiée d'utilisateurs virtuels est la plus sécurisée des 3.

Pour ne traiter que celle-ci, appliquez dans vsftpd.conf :

anonymous\_enable=NO              \# Accès anonyme interdit

Editez ensuite le fichier de configuration PAM :

\[srvlan@srvlan:~$\] sudo nano /etc/pam.d/vsftpd

et commentez ces 4 lignes dédiées utilisateurs locaux :

\# @include common-account
# @include common-session
# @include common-auth
# auth      required      pam\_shells.so

La 2ème ligne de ce fichier fait référence au fichier /etc/ftpusers. Pour info, celui-ci contient une liste par défaut d'utilisateurs locaux n'ayant pas le droit de se connecter sur le service FTP.

On pourrait encore augmenter la sécurité _(changement de n° de port FTP, filtrage IP, adresses IP virtuelles, etc...)_ mais la connexion authentifiée d'utilisateurs virtuels est un bon début.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Repos mérité, respirez !  
Le mémento 10.11 propose de rendre  
le réseau plus cohérent avec entre  
autres la création d'un DNS split.

[Mémento 10.11](https://familleleloup.no-ip.org/dns-split-debian11/)
