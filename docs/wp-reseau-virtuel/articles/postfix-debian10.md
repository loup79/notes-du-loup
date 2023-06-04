---
title: "E-mail / Debian10 : 1/3"
date: "2022-04-14"
categories: 
  - "archives"
---

## Mémento 11.1 - Postfix et Dovecot

Le serveur de courrier sera installé sur srvdmz.

L'installation de base inclura l'utilisation des logiciels Postfix, Dovecot, Mail et Postfixadmin.

### 1 - Préambule

Documentation de Postfix en français [sur cette page](https://postfix.traduc.org).

#### _1.1 - Rôle_ du serveur de courrier

Il consiste tout simplement à transmettre des messages électroniques ou e-mails à destination.

#### _1.2 - Principe de fonctionnement_

Le traitement du courrier respecte les étapes suivantes :

1) Départ depuis le client e-mail MUA de l'expéditeur  
\> Un serveur d'envoi MTA _(SMTP)_

2) Acheminement au travers d'un ou plusieurs MTA  
\> Un serveur de réception MDA _(POP/IMAP)_

3) Demande de relève depuis le MUA du destinataire  
\> Le serveur de réception MDA

4) Acheminement final depuis le MDA  
\> Le MUA du destinataire

Le MUA _(Mail User Agent)_ ou client e-mail s'occupe d'émettre et relever le courrier.

Le MTA _(Mail Transfer Agent)_ gère le transfert du courrier émis jusqu'à sa destination.

Le MDA _(Mail Delivery Agent)_ stocke le courrier reçu en attendant une relève par le destinataire.

Le protocole SMTP est utilisé par les MUA et MTA pour envoyer le courrier à destination.  
Les protocoles POP/IMAP le sont par les MDA et MUA pour relever le courrier.

Liste des logiciels auxquels vous ferez appel :  
\- Le trio apache + php + mysql déjà présent sur srvdmz.  
\- postfix comme MTA ou serveur SMTP.  
\- dovecot comme MDA ou serveur POP/IMAP.  
\- mail comme MUA en mode console.  
\- thunderbird comme MUA en mode graphique.  
\- postfixadmin comme interface Web d'administration.

Vous l'aurez compris, la mise en place d'un serveur de courrier est relativement compliquée.

Schéma récapitulant l'envoi d'un courrier :

![Synoptique - Postfix : Envoi d'un courrier de A vers B et B vers A](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfix-envoi-courrier.webp)

### 2 - Installation de Postfix, Dovecot et Mail

#### _2.1 - Postfix (MTA ou serveur SMTP)_

Au préalable, mettez à jour la distribution Debian :

\[srvdmz@srvdmz:~$\] sudo apt update             \# MAJ paquets
\[srvdmz@srvdmz:~$\] sudo apt upgrade     \# MAJ distribution
\[srvdmz@srvdmz:~$\] sudo reboot

et installez le paquet postfix :

\[srvdmz@srvdmz:~$\] sudo apt install postfix

Une fenêtre Postfix Configuration s'ouvre :  
\> OK > Site Internet > OK  
\> OK > Nom du courrier > loupvirtuel.fr > OK

Editez ensuite son fichier de configuration main.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et modifiez myhostname et mydestination comme suit :

myhostname = srvdmz.loupvirtuel.fr 
myorigin = /etc/mailname
mydestination = $myhostname, loupvirtuel.fr, srvdmz, \\
                              localhost.loupvirtuel.fr, localhost

Le caractère \\ indique d'écrire le tout sur une seule ligne.

\- myhostname = Nom FQDN du serveur Postfix  
\- myorigin = Nom du domaine sortant soit loupvirtuel.fr  
\- $myhostname = Valeur de myhostname  
\- mydestination = Noms des domaines entrant acceptés

Vérifiez par sécurité la bonne configuration de Postfix :

\[srvdmz@srvdmz:~$\] sudo postfix check

Normal implique aucun retour, mais Postfix renvoie :

```
postfix/postfix-script: warning: symlink leaves directory: /etc/postfix/./makedefs.out
```

Cet avertissement de configuration peut être ignoré.

Redémarrez Postix :

\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

et testez une connexion sur celui-ci avec la Cde telnet :

\[srvdmz@srvdmz:~$\]  telnet localhost 25        \# 25 = port SMTP

Retour observé :

```
Trying ::1...
Connected to localhost.
Escape character is '^]'.
220 srvdmz.loupvirtuel.fr ESMTP Postfix (Debian/GNU)
```

Puis touches Ctrl droite + AltGr + \] > prompt telnet > quit

_Nota : L'installation a généré un utilisateur postfix et un groupe de même nom sur srvdmz._

#### _2.2 - Dovecot (MDA ou serveur POP/IMAP)_

Installez les paquets dovecot suivants :

\[srvdmz@srvdmz:~$\] sudo apt install dovecot-mysql \\
                                       dovecot-imapd dovecot-pop3d

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Etendez les possibilités de filtrage du courrier entrant en installant le plugin sieve :

\[srvdmz@srvdmz:~$\] sudo apt install dovecot-managesieved

Les filtres fournis par le plugin permettront de traiter un courrier entrant en fonction d’un certain nombre de règles _(scripts)_ définies par le destinataire.

Les règles seront enregistrées et traitées sur le serveur.

Le plugin permettra entre autres de gérer de la réponse automatique en cas d'absence, de réexpédier des courriers dès leur réception sur le serveur, etc...

Les règles sont généralement définies via des interfaces de gestion respectant le protocole de filtrage telles que celles proposées par les plugins Sieve de Thunderbird et RoundCube_._

Activez maintenant la prise en compte du plugin :

\[srvdmz@srvdmz:~$\] sudo systemctl restart dovecot

et testez une connexion POP3 sur Dovecot :

\[srvdmz@srvdmz:~$\] telnet localhost 110    \# 110 = port POP3

Retour observé :

```
Trying ::1...
Connected to localhost.
Escape character is '^]'.
+OK Dovecot (Debian) ready.
```

Puis touches Ctrl droite + AltGr + \] > prompt telnet > quit

Testez également le résultat d'une connexion IMAP :

\[srvdmz@srvdmz:~$\] telnet localhost 143   \# 143 = port IMAP4

_Nota : L'installation a généré un utilisateur dovecot et un groupe de même nom sur srvdmz._

#### _2.3 - Mail (MUA ou client en mode console)_

Installez le paquet mailutils :

\[srvdmz@srvdmz:~$\]  sudo apt install mailutils

### 3 - Service DNS et courriers adressés à root

#### _3.1 - Ajout d'enregistrements type MX/CNAME_

Vous allez ajouter un enregistrement de type MX _(Mail eXchanger)_ qui associera le domaine loupvirtuel.fr au serveur de courrier installé sur srvdmz.

Cet enregistrement permettra de déterminer vers quel serveur un courrier doit être acheminé lorsque le protocole SMTP est utilisé, ceci en associant la partie à droite de l'@ des adresses e-mail au serveur de courrier.

Vous en ajouterez aussi de type CNAME _(alias)_ pour les serveurs SMTP _(Postfix)_ et POP/IMAP _(Dovecot)_.

Editez pour cela le fichier DNS db.loupvirtuel.fr.directe :

\[srvdmz@srvdmz:~$\] cd /var/cache/bind
\[srvdmz@srvdmz:~$\] sudo nano db.loupvirtuel.fr.directe

et ajoutez ceci juste avant www IN CNAME srvdmz :

@    IN   MX 10   srvdmz.loupvirtuel.fr.
smtp  IN   CNAME   srvdmz
pop    IN   CNAME   srvdmz
imap  IN   CNAME   srvdmz

Faites de même avec db.loupvirtuel.fr.directe.externe.

Redémarrez les services DNS et Postfix :

\[srvdmz@srvdmz:~$\] sudo systemctl restart bind9
\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

ainsi que le DNS sur srvlan.

Puis testez le bon fonctionnement du DNS comme suit :

\[srvdmz@srvdmz:~$\] ping smtp.loupvirtuel.fr
\[srvdmz@srvdmz:~$\] ping pop.loupvirtuel.fr
\[srvdmz@srvdmz:~$\] ping imap.loupvirtuel.fr
\[srvdmz@srvdmz:~$\] dig mx loupvirtuel.fr

[![Capture - Postfix : Retour de la Cde dig mx loupvirtuel.fr](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfix-enregistrement-mx-debian10-430x298.jpg "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfix-enregistrement-mx-debian10.jpg)

Postfix : Retour de la Cde dig mx loupvirtuel.fr

Les 4 Cdes ci-dessus doivent fonctionner depuis srvlan.

#### _3.2 - Redirection des courriers adressés à root_

La redirection des courriers se gère depuis le fichier Bdd /etc/aliases.db qui peut être mis à jour depuis le ficher texte /etc/aliases.

Editez ce dernier :

\[srvdmz@srvdmz:~$\] sudo nano /etc/aliases

et redirigez le courrier adressé à root en ajoutant ceci :

root: srvdmz

srvdmz étant l'utilisateur principal du serveur de courrier de nom d'hôte srvdmz.loupvirtuel.fr.

Terminez en mettant à jour le fichier Bdd aliases.db :

\[srvdmz@srvdmz:~$\] sudo newaliases

### 4 - Test d'envoi/réception de courriers

Dans l'immédiat, l'envoi d'un courrier vers un utilisateur quelconque tel clientmail-vm1 nécessite la création de celui-ci comme utilisateur local de l'hôte srvdmz.

Créez donc un utilisateur local UNIX clientmail-vm1:

\[srvdmz@srvdmz:~$\] sudo adduser clientmail-vm1

Puis envoyez un courrier de root vers clientmail-vm1:

\[srvdmz@srvdmz:~$\] su root
\[root@srvdmz:~$\]  mail clientmail-vm1@loupvirtuel.fr
Cc:                                                                        \>Touche Entrée
Subject: Test d'envoi depuis root                    \>Touche Entrée
Texte de mon premier courrier. Fin.   \>Touche Entrée
CTRL+D                                  \= Envoyer et quitter

et vérifiez la création dans /var/mail d'un fichier clientmail-vm1 stockant le contenu du courrier :

\[srvdmz@srvdmz:~$\] cd /var/mail
\[srvdmz@srvdmz:~$\] sudo cat clientmail-vm1

Connectez-vous maintenant en tant que clientmail-vm1 et lancez le MUA mail :

\[srvdmz@srvdmz:~$\] su clientmail-vm1
\[clientmail-vm1@srvdmz:~$\] mail

Relevez ensuite le courrier avec la Cde t 1 suivie de quit :

[![Capture - Postfix : Relève de l'e-mail n°1 adressé à clientmail-vm1](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfix-premier-courrier-debian10-430x224.jpg "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfix-premier-courrier-debian10.jpg)

Postfix : Relève de l'e-mail n°1 adressé à clientmail-vm1

Reconnectez-vous en tant que srvdmz avec la Cde exit et vérifiez la création dans /home/clientmail-vm1 d'un fichier mbox stockant le contenu du courrier relevé.

Vérifiez le vidage consécutif du contenu de ce courrier dans le fichier /var/mail/clientmail-vm1.

Finissez en envoyant un e-mail de clientmail-vm1 vers root et vérifiez la bonne redirection du courrier en relevant celui-ci depuis l'utilisateur srvdmz.

### 5 - Ajout des couches de sécurité SSL et SASL

#### _5.1 - Couche SSL authentification/chiffrement_

\- SSL/TLS  
Couche Secure Sockets Layer/Transport Layer Security.

Cette couche fournit des authentifications basées sur des certificats ainsi que le chiffrement des sessions de transport. Une session chiffrée protège le contenu des messages SMTP ainsi que celui des authentifications SASL.

Vous utiliserez pour Postfix les certificats loupvirtuel.crt et loupvirtuel.key créés au [§ 5 du mémento LAMP ...](/https-et-wordpress/#5_-_Mise_en_place_du_protocole_HTTPS)

Ces 2 fichiers peuvent servir à toutes applications faisant appel au chiffrage SSL.

#### _5.2 - Configuration SSL de Postfix_

Indiquez-lui le chemin des certificats en éditant main.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et en modifiant les lignes ci-dessous comme suit :

smtpd\_tls\_cert\_file=/etc/ssl/loupvirtuel.crt
smtpd\_tls\_key\_file=/etc/ssl/loupvirtuel.key

Activez la modification :

\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

#### _5.3 - Couche SASL authentification utilisateurs_

\- SASL  
Couche Simple Authentication and Security Layer.

Cette couche, utilisable avec des protocoles fonctionnant en mode connecté comme le SMTP, permet d'authentifier les utilisateurs cherchant à envoyer des messages par le biais du serveur de courrier et ainsi éviter que celui-ci ne soit notamment exploité pour diffuser du spam.

Installez les paquets suivants :

\[srvdmz@srvdmz:~$\] sudo apt install sasl2-bin db-util

Les dépendances libsasl2-2 et libsasl2-modules sont déjà présentes sur srvdmz.

Pour activer automatiquement le service SASL au boot du système, éditez le fichier saslauthd :

\[srvdmz@srvdmz:~$\] sudo nano /etc/default/saslauthd

et changez la valeur du paramètre START comme suit :

START=yes

Démarrez le service :

\[srvdmz@srvdmz:~$\] sudo systemctl start saslauthd

#### _5.4 - Configuration SASL de Postfix et Dovecot_

Vous allez maintenant configurer le système afin que Postix utilise l'authentification SMTP en faisant appel au support SASL fourni de base avec Dovecot.

a) Configuration côté Dovecot

Editez, le fichier 10-master.conf :

\[srvdmz@srvdmz:~$\] cd /etc/dovecot/conf.d
\[srvdmz@srvdmz:~$\] sudo nano 10-master.conf

et modifiez la section Postfix smtp-auth comme suit :

\# Postfix smtp-auth  
unix\_listener /var/spool/postfix/private/auth {  
mode = 0666  
user = postfix  
group = postfix  
}

Editez ensuite le fichier 10-auth.conf :

\[srvdmz@srvdmz:~$\] sudo nano 10-auth.conf

et modifiez la valeur de auth\_mechanisms comme suit :

auth\_mechanisms = plain login

Redémarrez Dovecot :

\[srvdmz@srvdmz:~$\] sudo systemctl restart dovecot

b) Configuration côté Postfix

Editez le fichier de configuration main.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et ajoutez les lignes suivantes en fin de fichier :

\# Activation du protocole SASL partie serveur SMTP de Postfix
smtpd\_sasl\_type = dovecot
smtpd\_sasl\_path = private/auth
smtpd\_sasl\_auth\_enable = yes
smtpd\_recipient\_restrictions = permit\_sasl\_authenticated, \\
                       permit\_mynetworks,reject\_unauth\_destination

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Redémarrez Postfix :

\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

#### _5.5 - Test de la configuration SSL et SASL_

Vérifiez la prise en compte des 2 protocoles de sécurisation à l'aide de la Cde suivante :

\[srvdmz@srvdmz:~$\] telnet localhost 25    \# 25 = n° port SMTP

Retour :

```
Trying ::1...
Connected to localhost.
Escape character is '^]'.
220 srvdmz.loupvirtuel.fr ESMTP Postfix (Debian/GNU)
```

Entrez sous la ligne 220 srvdmz... l'instruction suivante :

ehlo srvdmz.loupvirtuel.fr

et observez le retour :

```
250-srvdmz.loupvirtuel.fr
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-AUTH PLAIN LOGIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```

Doivent être présents STARTTLS et AUTH PLAIN LOGIN.

Fermez la connexion à l'aide de la Cde quit.

#### _5.6 - Chiffrage TLS des connexions SMTP_

SASL avec son MDP diffusé en clair sur le réseau, impose par sécurité, de chiffrer la phase d'authentification et également d'utiliser le port de soumission 587 à la place du port 25.

Vous allez donc forcer les clients MUA du réseau comme l'application thunderbird à établir des sessions SMTP entièrement chiffrées TLS _(pas uniquement l'authentification SASL)._

Editez pour cela le fichier de configuration master.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/master.cf

et activez le port 587 en décommentant cette ligne :

submission inet n - - - - smtpd

Editez ensuite le fichier principal main.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et ajoutez les 3 lignes suivantes à la fin de celui-ci :

\# Session SMTP et authentification SASL
# issues des clients e-mail MUA forcées TLS
smtpd\_tls\_security\_level = encrypt 

Les échanges entre MUA et Postix seront ainsi cryptés.

Relancez Postfix pour traiter la nouvelle configuration :

\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

### 6 - Envoi de courriers vers Internet _(relais SMTP)_

Vous accédez généralement à Internet au travers d'une box de FAI telle celle d'Orange ou autre.

Malgré le côté fictif du nom de domaine loupvirtuel.fr, vous allez envoyer du courrier au travers d'Internet en demandant à Postfix d'utiliser le serveur de courrier du FAI comme relais SMTP.

La configuration ci-dessous concerne le FAI Orange qui accepte de jouer le rôle de relais SMTP après validation d'une demande par authentification SASL chiffrée TLS sur le port 587.

Commencez par créer un fichier texte sasl\_passwd :

\[srvdmz@srvdmz:~$\] cd /etc/postfix
\[srvdmz@srvdmz:~$\] sudo nano sasl\_passwd

et entrez le contenu suivant :

\[smtp.orange.fr\]:587 utilisateur\_orange:MDP\_orange

Chiffrez ensuite le contenu :

\[srvdmz@srvdmz:~$\] sudo postmap hash:sasl\_passwd

Un fichier Bdd sasl\_passwd.db est alors créé.

Protégez le fichier texte en modifiant ses permissions :

\[srvdmz@srvdmz:~$\] sudo chmod 600 sasl\_passwd

Editez maintenant le fichier main.cf :

\[srvdmz@srvdmz:~$\] sudo nano main.cf

Modifiez son paramètre relayhost comme suit :

relayhost = \[smtp.orange.fr\]:587

et ajoutez ces lignes à la fin du fichier :

\# Activation du protocole SASL partie client SMTP de Postfix
smtp\_sasl\_password\_maps = hash:/etc/postfix/sasl\_passwd
smtp\_sasl\_auth\_enable = yes
smtp\_sasl\_security\_options = noanonymous

# Désactivation du smtputf8 non géré par le relais Orange
smtputf8\_enable = no

Redémarrez Postfix :

\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

L'envoi vers Internet nécessite, pour éviter l'alerte Emetteur invalide, que l'adresse d'émission du courrier contienne un nom de domaine connu tel que xxx@orange.fr, xxx@yahoo.fr, etc...

Ci-dessous, l'exemple d'un envoi depuis une adresse mail Orange vers une adresse mail Google :

\[srvdmz@srvdmz:~$\] mail -s "Test relayhost" -a "From: xxx@orange.fr" xxx@gmail.com
Cc: <Entrée>
Texte du courrier à entrer ici. <Entrée>
CTRL+D
\[srvdmz@srvdmz:~$\]

Testez selon votre configuration, cela doit fonctionner.

### 7 - Interface Web d'administration Postfixadmin

L'interface demande, pour fonctionner, de modifier la configuration des serveurs Apache-PHP, MySQL, Postfix et Dovecot.

Elle permettra de gérer le domaine loupvirtuel.fr et les utilisateurs du serveur de courrier sans être obligé de créer sur srvdmz un compte Unix par utilisateur. Le domaine et les utilisateurs seront virtuels et contenus dans une Bdd.

La gestion multi-domaines sera également possible.

Par sécurité, la liaison entre l'interface Web et le serveur Apache sera chiffrée SSL/TLS.

#### _7.1 - Installation_

Rappel : Le module ssl pour Apache a déjà été activé au [§ 5.4 du mémento LAMP HTTPS ...](/https-et-wordpress/#54_-_Hote_virtuel_pour_lacces_au_site_Web_et_test_HTTPS)

L'installateur de postfixadmin lira le fichier debian.cnf pour se connecter sur MySQL mais la désactivation du plugin unix\_socket pour root configurée dans le [§ 3 du mémento LAMP HTTPS ...](/site-web-php-mysql/#3_-_Installation_du_gestionnaire_de_Bdd_Adminer) empêche la connexion.

Pour assurer l'installation, éditez ce fichier MySQL :

\[srvdmz@srvdmz:~$\] sudo nano /etc/mysql/debian.cnf

et modifiez-le comme suit :

\# Automatically generated for Debian scripts. DO NOT TOUCH!
 \[client\]
 host     = localhost
 user     = root
 password = MDP en clair de l'utilisateur root pour mySQL
 socket   = /var/run/mysqld/mysqld.sock
 \[mysql\_upgrade\]
 host     = localhost
 user     = root
 password = MDP en clair de l'utilisateur root pour mySQL
 socket   = /var/run/mysqld/mysqld.sock
 basedir  = /usr

Puis installez les paquets de postfixadmin :

\[srvdmz@srvdmz:~$\] sudo apt install postfixadmin postfix-mysql

Une fenêtre Configuration de Postfixadmin s'ouvre :  
\- Faut-il conf... Bdd de ... avec dbconfig-common ? > Oui  
\- MDP de c... MySQL pour postfixadmin > courriers > OK  
\- Confirmation du MDP > courriers > OK

L'installation reprend...

Une fenêtre Configuration de Postfixadmin s'ouvre :  
\- MDP administrateur Bdd > MDP du root MySQL > OK

L'installation continue jusqu'à son terme.

Il est possible de reconfigurer l'installation avec la Cde dpkg-reconfigure postfixadmin.

Créez à présent un fichier config.local.php :

\[srvdmz@srvdmz:~$\] cd /etc/postfixadmin
\[srvdmz@srvdmz:~$\] sudo nano config.local.php

et entrez le contenu PHP suivant :

<?php
$CONF\['default\_language'\] = 'fr';
$CONF\['admin\_email'\] = 'postmaster@loupvirtuel.fr';
?>

Déclarez pour ce fichier le lien symbolique suivant :

\[srvdmz@srvdmz:~$\] sudo ln -s \\
                /etc/postfixadmin/config.local.php \\
               /usr/share/postfixadmin/config.local.php

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Editez ensuite le fichier dbconfig.inc.php :

\[srvdmz@srvdmz:~$\] sudo nano dbconfig.inc.php

et modifiez la valeur du paramètre $dbtype comme suit :

$dbtype='mysqli';

Pour finir, créez un lien symbolique du dossier Web postfixadmin vers la racine Web de Apache :

\[srvdmz@srvdmz:~$\] sudo ln -s /usr/share/postfixadmin \\
                                       /var/www/postfixadmin

Ajoutez un dossier Web vide de nom templates\_c :

\[srvdmz@srvdmz:~$\]  sudo mkdir /usr/share/postfixadmin/templates\_c

et attribuez au dossier postfixadmin les mêmes permissions que celles du dossier wordpress.

\[srvdmz@srvdmz:~$\]  sudo chown -R www-data:www-data /usr/share/postfixadmin

#### _7.2 - Hôte virtuel et redirection HTTPS_

Corrigez la configuration de l'hôte virtuel en éditant le fichier Apache postfixadmin.conf :

\[srvdmz@srvdmz:~$\]  cd /etc/apache2/conf-available
\[srvdmz@srvdmz:~$\]  sudo nano postfixadmin.conf

et en modifiant la ligne Alias ... comme suit :

Alias /postfixadmin /usr/share/postfixadmin/public

La redirection HTTP du domaine loupvirtuel.fr vers HTTPS est assurée depuis cette section située au début du fichier /etc/apache2/sites-available/loupvirtuel.conf :

<VirtualHost \*:80>
ServerName loupvirtuel.fr
ServerAlias www.loupvirtuel.fr srvdmz.loupvirtuel.fr
Redirect permanent / https://loupvirtuel.fr/
</VirtualHost>

Si vous avez exécuté le [mémento 8.2](/https-et-wordpress/#5_-_Mise_en_place_du_protocole_HTTPS), les 3 URL HTTP ci-dessous seront redirigées automatiquement vers HTTPS sans faire l'objet d'une alerte de sécurité.

\- http://loupvirtuel.fr/postfixadmin  
\- http://www.loupvirtuel.fr/postfixadmin  
\- http://srvdmz.loupvirtuel.fr/postfixadmin

Redémarrez le service Apache :

\[srvdmz@srvdmz:~$\]  sudo systemctl restart apache2

Ensuite, depuis srvlan, lancez l'URL suivante :  
https://loupvirtuel.fr/postfixadmin/setup.php

Une fois la page setup.php ouverte, actualisez-la pour obtenir le contenu ci-dessous, celui-ci précisant si toutes les dépendances sont installées et configurées :

[![Capture - Postfix : Page setup.php de Postfixadmin](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-page-setup-debian10-430x409.jpg "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-page-setup-debian10.jpg)

Postfixadmin - Page setup.php

Fournissez, comme demandé, un Setup password :  
\> Entrez le MDP mailadmin31 > Confirmez le MDP  
\> Cliquez sur le bouton Generate password hash

\> La page Web s'actualise et montre le hash du MDP  
\> Gardez la page ouverte

Editez à présent, sur srvdmz, le fichier config.local.php :

\[srvdmz@srvdmz:~$\] cd /etc/postfixadmin
\[srvdmz@srvdmz:~$\] sudo nano config.local.php

et ajoutez le paramètre ci-dessous comme suit :

$CONF\['setup\_password'\] = 'Entrez le hash affiché sur srvlan';

Revenez sur srvlan et créez, comme demandé sur la page Web, le compte super-administrateur :  
\> Setup password > mailadmin31  
\> Administrateur > postmaster@loupvirtuel.fr  
\> Mot de passe > postmaster31 > Confirmez  
\> Cliquez sur le bouton Ajouter un administrateur

Le texte de validation suivant apparaît si tout est OK :

     L'administrateur postmaster@loupvirtuel.fr a été ajouté !

Fermez la page Web et entrez maintenant l'URL :  
https://loupvirtuel.fr/postfixadmin/

La page de connexion de PostfixAdmin doit s'ouvrir :  
\- Adresse > postmaster@loupvirtuel.fr  
\- Mot de passe > postmaster31

[![Capture - Postfixadmin : Vue de la page de connexion ](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-page-login-debian10-430x197.jpg "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-page-login-debian10.jpg)

Postfixadmin - Vue de la page de connexion

Créez loupvirtuel.fr depuis l'onglet Liste des domaines :  
\> Ajouter un domaine  
\> Champ Domaine = loupvirtuel.fr  
\> Décochez Ajouter les alias par défaut  
\> Bouton Ajouter un domaine

[![Capture - Postfixadmin : Page montrant le domaine virtuel créé](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-domaine-virtuel-debian10-430x153.jpg "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-domaine-virtuel-debian10.jpg)

Postfixadmin - Page montrant le domaine virtuel créé

Créez ces 5 comptes mail depuis Liste des virtuels :  
\> Ajouter un compte courriel  
\> Champ Nom d'utilisateur = clientmail-vm1  
\> Mot de passe > Votre MDP  
\> Mot de passe (confirmation) > Votre MDP  
\> Nom > debian10-vm1  
\> Bouton Ajouter un compte courriel

[![Capture - Postfixadmin : Page montrant les comptes virtuels créés](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-comptes-virtuels-debian10-430x239.jpg "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-comptes-virtuels-debian10.jpg)

Postfixadmin - Page montrant les comptes virtuels créés

ainsi qu'un alias de root vers postmaster :  
\> Ajouter un alias  
\> Champ Alias = root  
\> Champ A = postmaster@loupvirtuel.fr  
\> Bouton Ajouter un alias

[![Capture - Postfixadmin : Page montrant l'alias créé](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-alias-root-debian10-430x220.jpg "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/postfixadmin-alias-root-debian10.jpg)

Postfixadmin - Page montrant l'alias créé

#### _7.3 - Accès de Postfix sur Bdd Postfixadmin_

Revenez sur srvdmz.

Vous allez créer un utilisateur local vmail qui n'aura pas de MDP associé et se verra donc interdit de connexion.

Un dossier /var/mail/vmail affecté à cet utilisateur et à un groupe de même nom servira de racine aux comptes e-mail virtuels.

Commencez par créer le groupe/utilisateur vmail :

\[srvdmz@srvdmz:~$\] sudo groupadd -g 5000 vmail
\[srvdmz@srvdmz:~$\] sudo useradd -g vmail -u 5000 vmail -d /home/vmail -s /bin/false

GID/UID 5000 = identifiants groupe/utilisateur.

et le répertoire ou seront stockés les e-mails :

\[srvdmz@srvdmz:~$\] sudo mkdir -p /var/mail/vmail
\[srvdmz@srvdmz:~$\] cd /var/mail
\[srvdmz@srvdmz:~$\] sudo chown -R 5000:5000 vmail
\[srvdmz@srvdmz:~$\] sudo chmod -R 770 vmail

Créez, pour l'accès à la Bdd postfixadmin, l'utilisateur MySQL postfix avec le MDP postfixmysql :

\[srvdmz@srvdmz:~$\] mysql -u root -p
Enter password: Votre MDP comme utilisateur root de MySQL

MariaDB \[(none)\]> grant all privileges on postfixadmin.\* to 'postfix'@'192.168.4.2' identified by 'postfixmysql'; 

Query OK, 0 rows affected (0.00 sec)
MariaDB \[(none)\]> quit
Bye

Créez le fichier qui assurera la recherche des domaines virtuels déclarés sous postfixadmin :

\[srvdmz@srvdmz:~$\] cd /etc/postfix
\[srvdmz@srvdmz:~$\] sudo nano mysql\_virtual\_mailbox\_domains\_maps.cf

et entrez le contenu ci-dessous :

user = postfix
password = postfixmysql
hosts = 192.168.4.2                        \# bind address de mysql       
dbname = postfixadmin  
query = SELECT domain FROM domain WHERE domain='%s' AND active = 1

Déclarez ensuite ce fichier dans /etc/postfix/main.cf :

\[srvdmz@srvdmz:~$\] sudo postconf -e \\
                                       virtual\_mailbox\_domains=\\
   mysql:/etc/postfix/mysql\_virtual\_mailbox\_domains\_maps.cf  

Le caractère \\ indique d'écrire le tout sur une seule ligne.

et testez l'existence de loupvirtuel.fr dans postfixadmin :

\[srvdmz@srvdmz:~$\] sudo postmap -q loupvirtuel.fr mysql:\\
           /etc/postfix/mysql\_virtual\_mailbox\_domains\_maps.cf

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Si OK, la Cde retourne loupvirtuel.fr, sinon rien.

Créez le fichier qui assurera la recherche des comptes mail virtuels déclarés sous postfixadmin :

\[srvdmz@srvdmz:~$\] sudo nano mysql\_virtual\_mailbox\_maps.cf

et entrez les lignes ci-dessous :

user = postfix
password = postfixmysql
hosts = 192.168.4.2                            \# bind address de mysql
dbname = postfixadmin
query = SELECT maildir FROM mailbox WHERE username='%s' AND active = 1

Déclarez ensuite ce fichier dans /etc/postfix/main.cf :

\[srvdmz@srvdmz:~$\] sudo postconf -e \\
                                       virtual\_mailbox\_maps=\\
           mysql:/etc/postfix/mysql\_virtual\_mailbox\_maps.cf

Le caractère \\ indique d'écrire le tout sur une seule ligne.

et testez l'existence de srvlan dans postfixadmin :

\[srvdmz@srvdmz:~$\] sudo postmap -q srvlan@loupvirtuel.fr \\
                   mysql:/etc/postfix/mysql\_virtual\_mailbox\_maps.cf

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Si OK, la Cde retourne loupvirtuel.fr/srvlan/, sinon rien.

Créez le fichier qui assurera la recherche des alias virtuels déclarés sous postfixadmin :

\[srvdmz@srvdmz:~$\] sudo nano mysql\_virtual\_mailbox\_aliases\_maps.cf

et entrez les lignes ci-dessous :

user = postfix
password = postfixmysql
hosts = 192.168.4.2                         \# bind address de mysql
dbname = postfixadmin
query = SELECT goto FROM alias WHERE address='%s' AND active = 1

Déclarez ensuite ce fichier dans /etc/postfix/main.cf :

\[srvdmz@srvdmz:~$\] sudo postconf -e \\
                                       virtual\_alias\_maps=mysql:\\
       /etc/postfix/mysql\_virtual\_mailbox\_aliases\_maps.cf

Le caractère \\ indique d'écrire le tout sur une seule ligne.

et testez l'existence d'un alias pour root :

\[srvdmz@srvdmz:~$\] sudo postmap -q root@loupvirtuel.fr \\
     mysql:/etc/postfix/mysql\_virtual\_mailbox\_aliases\_maps.cf

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Si OK, la Cde retourne postmaster@loupvirtuel.fr.

Sécurisez les droits d'accès sur les 3 fichiers de recherche dont les MDP sont écrits en clair :

\[srvdmz@srvdmz:~$\] cd /etc/postfix
\[srvdmz@srvdmz:~$\] sudo chgrp postfix mysql\_\*.cf
\[srvdmz@srvdmz:~$\] sudo chmod u=rw,g=r,o= mysql\_\*.cf

Editez maintenant le fichier main.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et ajoutez ceci en fin de fichier juste au dessus des lignes déclarant les 3 fichiers de recherche :

\# Paramètres pour utiliser domaines et utilisateurs virtuels
local\_transport = virtual
virtual\_gid\_maps = static:5000
virtual\_uid\_maps = static:5000
virtual\_mailbox\_base = /var/mail/vmail

_( ci-dessous lignes déclarant les 3 fichiers de recherche)_
virtual\_mailbox\_domains = mysql:/etc/postfix/\\ 
mysql\_virtual\_mailbox\_domains\_maps.cf
virtual\_mailbox\_maps = mysql:/etc/postfix/\\
mysql\_virtual\_mailbox\_maps.cf
virtual\_alias\_maps = mysql:/etc/postfix/\\
mysql\_virtual\_mailbox\_aliases\_maps.cf 

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Le domaine loupvirtuel.fr créé dans Postfixadmin impose de supprimer sa référence dans main.cf, en effet un domaine virtuel ne doit jamais se trouver dans la liste du paramètre mydestination.

Editez pour cela main.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et modifiez le paramètre mydestination comme suit :

mydestination = srvdmz, localhost

Terminez en redémarrant Postfix :

\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

#### _7.4 - Accès de Dovecot sur Bdd Postfixadmin_

Par défaut, Dovecot utilise les utilisateurs réels du système et non les utilisateurs virtuels.

Pour corriger cela, éditez le fichier 10-auth.conf :

\[srvdmz@srvdmz:~$\] cd /etc/dovecot
\[srvdmz@srvdmz:~$\] sudo nano conf.d/10-auth.conf

et modifiez le groupe !include comme suit :

#!include auth-system.conf.ext  
!include auth-sql.conf.ext  
#!include auth-ldap.conf.ext  
#!include auth-passwdfile.conf.ext  
#!include auth-checkpassword.conf.ext  
#!include auth-vpopmail.conf.ext  
#!include auth-static.conf.ext

Pour traiter l'accès SQL, éditez auth-sql.conf.ext :

\[srvdmz@srvdmz:~$\] sudo nano conf.d/auth-sql.conf.ext

Commentez la 2ème section userdb puis modifiez la 3ème section comme suit :

userdb {    
     driver = static    
     args = uid=vmail gid=vmail home=/var/mail/vmail/%d/%n    
     }

Pour traiter le dossier des e-mails, éditez 10-mail.conf :

\[srvdmz@srvdmz:~$\] sudo nano conf.d/10-mail.conf

et modifiez le paramètre mail\_location comme suit :

mail\_location = maildir:/var/mail/vmail/%d/%n/Maildir

Les e-mails sont ainsi déclarés stockés sous la forme :  
/var/mail/vmail/\[domaine\]/\[utilisateur\]/Maildir

Pour configurer l'authentification ssl, éditez 10-ssl.conf :

\[srvdmz@srvdmz:~$\] sudo nano conf.d/10-ssl.conf

et modifiez les 3 paramètres ci-dessous comme suit :

ssl = required
ssl\_cert = </etc/ssl/loupvirtuel.crt
ssl\_key = </etc/ssl/loupvirtuel.key

Les certificats SSL sont ceux déclarés au [§ 5.2](/serveur-de-courrier-postfix-partie-1/#52_-_Configuration_SSL_de_Postfix).

Editez également de nouveau le fichier 10-auth.conf :

\[srvdmz@srvdmz:~$\] sudo nano conf.d/10-auth.conf

et décommentez la ligne disable\_plaintext\_auth = yes.

Pour configurer le LDA, éditez 15-lda.conf :

\[srvdmz@srvdmz:~$\] sudo nano conf.d/15-lda.conf

Décommentez le paramètre postmaster\_address puis modifiez-le comme suit :

postmaster\_address = postmaster@loupvirtuel.fr

Modifiez aussi le bloc protocol lda comme ci-dessous :

protocol lda {  
     # Space separated list of plugins to load (default is glob...).  
     mail\_plugins = $mail\_plugins sieve 
     }

Ceci active le plugin de filtrage des e-mails soit sieve.

Pour la connexion sql, éditez dovecot-sql.conf.ext :

\[srvdmz@srvdmz:~$\] sudo nano dovecot-sql.conf.ext

et ajoutez à la fin de celui-ci le bloc suivant :

driver = mysql

connect = host=192.168.4.2 dbname=postfixadmin user=postfix password\=postfixmysql

password\_query = SELECT username,domain,password FROM mailbox WHERE username='%u';

Puis changez ses droits pour le password écrit en clair :

\[srvdmz@srvdmz:~$\] sudo chown root:root dovecot-sql.conf.ext
\[srvdmz@srvdmz:~$\] sudo chmod go= dovecot-sql.conf.ext

Changez les droits de dovecot.conf pour que Dovecot soit lancé en tant qu'utilisateur vmail :

\[srvdmz@srvdmz:~$\] sudo chgrp vmail dovecot.conf
\[srvdmz@srvdmz:~$\] sudo chmod g+r dovecot.conf

Terminez en redémarrant Dovecot :

\[srvdmz@srvdmz:~$\] sudo systemctl restart dovecot

#### _7.5 - Utilisation par Postfix du LDA de Dovecot_

Vous allez demander à Postfix de ne pas utiliser son propre LDA _(Local Delivery Agent)_ pour trier les e-mails dans leurs dossiers de destination respectifs mais plutôt celui de Dovecot plus performant et permettant de profiter des avantages du plugin Sieve.

Pour cela éditez le fichier master.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/master.cf

et ajoutez à la fin de celui-ci les lignes suivantes :

\# Connexion de Postfix à Dovecot
dovecot unix - n n - - pipe  
     flags=DRhu user=vmail:vmail argv=/usr/lib/dovecot/dovecot-lda -f ${sender} -d ${recipient}

Attention, la 3ème ligne doit être indentée de 2 espaces.

Editez ensuite le fichier main.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et ajoutez les lignes suivantes afin que Postifx exploite ce que vous venez de configurer :

\# Déclaration d'utilisation du LDA de Dovecot par Postfix  
virtual\_transport=dovecot  
dovecot\_destination\_recipient\_limit=1

Dovecot qui inclut maintenant un module de statistiques signalera une erreur si jamais celui-ci n’est pas défini dans sa configuration.

Pour éviter cela, éditez le fichier 10-master.conf :

\[srvdmz@srvdmz:~$\] cd /etc/dovecot
\[srvdmz@srvdmz:~$\] sudo nano conf.d/10-master.conf

et ajoutez ce bloc d'instructions :

service stats {
 unix\_listener stats-reader {
 user = vmail
 group = vmail
 mode = 0660
 }
 unix\_listener stats-writer {
 user = vmail
 group = vmail
 mode = 0660
 }
 }

Redémarrez à présent srvdmz :

\[srvdmz@srvdmz:~$\] sudo reboot

### 8 - Contrôle de Postfixadmin depuis le LAN

#### _8.1 - Envoi d'un courrier depuis le client Mail_

Envoyez un mail de clientmail-vm1 vers postmaster :

\[srvdmz@srvdmz:~$\] echo "Testé et approuvé" | mail \\
-s "Envoyé par clientmail-vm1" postmaster@loupvirtuel.fr \\
-a From:clientmail-vm1@loupvirtuel.fr  

Le caractère \\ indique d'écrire le tout sur une seule ligne.

et observez les contenus suivants du fichier syslog :

\[srvdmz@srvdmz:~$\] sudo cat /var/log/syslog | grep postfix

Retour pour Postfix :

```
postfix/pickup..2DDC: uid=.. from=<clientmail-vm1..>
postfix/cleanup..2DDC: message-id=<..2DDC@srvdmz..>
postfix/qmgr..2D.. from=<clientmail-vm1..>.. active)
postfix/pipe..2D.. to=<postmaster..>.. status=sent
postfix/qmgr..2D.. removed
```

Pas d'erreur signalée, courrier émis _(status sent)_.

\[srvdmz@srvdmz:~$\] sudo cat /var/log/syslog | grep dovecot

Retour pour Dovecot :

```
dovecot: lda(postmaster...)... msgid=<...2DDC...>: saved ...
```

Pas d'erreur signalée, courrier reçu et stocké _(saved)_.

#### _8.2 - Relève du courrier depuis Thunderbird_

A l'aide de l'outil Synaptic, installez sur srvlan les paquets thunderbird et thunderbird-l10n-fr.

Editez ensuite le menu openbox :

\[srvlan@srvlan:~$\] cd /home/srvlan/.config/openbox
\[srvlan@srvlan:~$\] nano menu.xml

et ajoutez sous l'item du Navigateur Chromium celui-ci :

<item label="Client mail \[thunderbird\]">
<action name="Execute">
<execute>thunderbird</execute>
</action>
</item>

Validez en rechargeant la configuration du menu depuis le menu contextuel d'openbox.

Copiez le certificat CA /etc/ssl/loupvirtuel.pem de srvdmz dans le dossier partagé du PC hôte.

Puis lancez Thunderbird et décalez sans la fermer la fenêtre Configurez votre adresse ... existante pour accédez au menu de celui-ci.

Menu de Thunderbird :  
\> Edition > Préférences > Vie privée et sécurité  
\> Certificats > Gérer les certificats...

Une fenêtre Gestionnaire de certificats s'ouvre :  
\> Onglet Autorités > Bouton Importer...  
\> Accédez au dossier partagé par le PC hôte  
\> Sélectionnez le certificat SSL loupvirtuel.pem > Ouvrir

Une fenêtre Téléchargement du certificat s'ouvre :  
\> Cochez Confirmer ... AC pour identifier ... web.  
\> Cochez Confirmer ... AC pour identifier ... courrier.  
\> OK > OK

Ceci évitera d'avoir à traiter une exception de sécurité due à l'usage d'un certificat auto-signé. En fait on simule un certificat vérifié par une autorité _(AC)_ publique.

Traitez la fenêtre Configurez votre adresse ... existante :  
\- Votre nom complet  
\> Le Postmaster du site loupvirtuel.fr  
\- Adresse électronique  
\> postmaster@loupvirtuel.fr  
\- Mot de passe  
\> Le Mdp que vous avez attribué à postmaster au [§ 7.2](/serveur-de-courrier-postfix-partie-1/#72_-_Hote_virtuel_et_redirection_HTTPS)

\> Configurer manuellement...

Appliquez la configuration ci-dessous :

[![Capture - thunderbird : Création du compte Postmaster](/wp-content/uploads/2021/03/thunderbird-configuration-compte-367x430.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/03/thunderbird-configuration-compte.jpg)

Thunderbird - Création du compte postmaster

Champ Serveur : Enlevez le . situé avant loupvirtuel.fr.

\> Re-tester et si tout est OK, affichage de la ligne :  
Les paramètres ... ont été trouvés ... le serveur indiqué

\> Terminé, postmaster est créé dans Thunderbird.

Cliquez ensuite sur Relever pour récupérer en lecture le courrier envoyé par clientmail-vm1 :

[![Capture - Thunderbird : Réception du courrier de clientmail-vm1](/wp-content/uploads/2021/03/thunderbird-reception-courrier-430x191.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/03/thunderbird-reception-courrier.jpg)

Thunderbird - Réception du courrier de clientmail-vm1

Vous pouvez à présent créer sur les debian10-vm\* et srvdmz les comptes appropriés en ayant préalablement installé Thunderbird.

Créez sur srvlan le compte srvlan@loupvirtuel.fr en complément du compte postmaster.

Effectuez des tests d'envois/réceptions entre les VM pour vérifier le bon fonctionnement.

Un envoi vers root doit parvenir à postmaster.

### 9 - Contrôle de Postfixadmin depuis Internet

Le domaine loupvirtuel.fr n'existant pas, il est possible de procéder ainsi avec une Livebox :

\- Enregistrez sur le site no-ip.com un nom de domaine gratuit de type xxx.no-ip.org et paramétrez l'option Mail associée MX Record avec ce même nom de domaine.

\- Créez ensuite le domaine virtuel xxx.no-ip.org sur Postfixadmin ainsi qu'un compte courriel virtuel x.y@xxx.no-ip.org de mot de passe y42.

\- Modifiez la configuration NAT de la LiveBox en dirigeant les demandes d'accès au port 25 vers l'adresse IP de la carte enp0s3 de srvsec qui est le point d'entrée du réseau local virtuel.

Ouvrez sur le pare-feu d'IPFire le port 25 vers srvdmz :

[![Capture - IPFire : Ouverture de port 25 d'Internet vers srvdmz](/wp-content/uploads/2021/03/ipfire-port-25-430x389.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/03/ipfire-port-25.jpg)

IPFire - Ouverture de port 25 d'Internet vers srvdmz

Procédez de même pour les ports 587 et 993 :

[![Capture - IPFire : Ports 25, 587 et 993 ouverts](/wp-content/uploads/2021/03/ipfire-serveur-courrier-430x76.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/03/ipfire-serveur-courrier.jpg)

IPFire - Ports 25, 587 et 993 ouverts

\- Créez maintenant sur le client MUA Thunderbird de srvlan un compte x.y@xxx.no-ip.org.  
Utilisez les mêmes paramètres IMAP et SMTP que les comptes créés précédemment.

Pour terminer, envoyez un e-mail depuis le MUA de votre smartphone vers l'adresse x.y@xxx.no-ip.org et relevez celui-ci depuis le Thunderbird de srvlan et inversement.

[![Capture - Thunderbird : Réception d'un courrier provenant d'Internet](/wp-content/uploads/2021/03/thunderbird-reception-internet-430x219.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/03/thunderbird-reception-internet.jpg)

Thunderbird - Réception d'un courrier issu d'Internet

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Ouf, fini.  
Postfix est maintenant opérationnel.  
La partie 2 vous attend pour gérer le  
traitement du SPAM et des VIRUS.

[Partie 2](/postfix-amavis-debian10/)
