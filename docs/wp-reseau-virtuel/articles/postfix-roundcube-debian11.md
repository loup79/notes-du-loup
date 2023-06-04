---
title: "E-mail / Debian 11 : 3/3"
date: "2022-10-25"
categories: 
  - "serveur-de-courrier"
---

## Mémento 11.31 - Webmail Roundcube

Le service Webmail fera appel au logiciel Roundcube qui est un client de messagerie au même titre que Thunderbird mais s'exécutant sur serveur.

Celui-ci servira d'interface entre le serveur de courrier et n'importe quel navigateur Web connecté à Internet ce qui le rend pratique pour une consultation de son courrier en situation de mobilité.

### 12 - Roundcube

#### _12.1 - Installation et configuration de base_

Installez le paquet Debian roundcube :

\[srvdmz@srvdmz:~$\] sudo apt install roundcube

Une fenêtre de configuration roundcube-core s'ouvre :  
\- Faut-il configurer ... roundcube ... -common ? > Oui  
\- MDP de connexion MySQL ... > roundcubemysql > OK  
\- Confirmation du MDP > roundcubemysql > OK

L'installation continue jusqu'à son terme.

Les dépendances ont été ajoutées automatiquement.

Les modules Apache proxy\_fcgi et setenvif ainsi que la configuration Apache php7.4-fpm sont déjà activés, voir [§ 7.1](/site-web-php-mysql/#3_-_Installation_du_gestionnaire_de_bases_de_donnees_Adminer) [du mémento 11.11](/postfix-debian11/#71_-_Installation).

Les fichiers ont été installés essentiellement dans :  
\- /etc/roundcube/ - /var/lib/roundcube/  
\- /usr/share/roundcube/ - /var/log/roundcube/

Une Bdd de nom roundcube a également été créée ainsi qu'un lien de configuration Apache de nom roundcube.conf dans /etc/apache2/conf-enabled/.

Editez ensuite le fichier de configuration config.inc.php :

\[srvdmz@srvdmz:~$\] cd /etc/roundcube
\[srvdmz@srvdmz:~$\] sudo nano config.inc.php

Modifiez le contenu de ces lignes comme suit :

$config\['default\_host'\] = 'ssl://loupvirtuel.fr';
$config\['smtp\_server'\] = 'tls://loupvirtuel.fr';
$config\['smtp\_port'\] = 587;
$config\['smtp\_user'\] = '%u'; 
$config\['smtp\_pass'\] = '%p';
$config\['product\_name'\] = 'Serveur Webmail Roundcube';

et ajoutez celles-ci à la fin du fichier :

\# Paramètres des connexions SSL/TLS
$config\['default\_port'\] = 993;       \# port IMAP

# SSL IMAP        \# Voir note dans le fichier defaults.inc.php
$config\['imap\_conn\_options'\] = array(
     'ssl' => array(
     'verify\_peer' => true,
     'verify\_depth' => 3,
     'cafile' => '/etc/ssl/loupvirtuel-ca.pem',
    ),
  );

# TLS SMTP       \# Voir note dans le fichier defaults.inc.php
$config\['smtp\_conn\_options'\] = array(
     'ssl' => array(
     'verify\_peer' => true,
     'verify\_depth' => 3,
     'cafile' => '/etc/ssl/loupvirtuel-ca.pem',
    ),
  );

# Ajout auto du domaine aux créations d'utilisateurs
$config\['mail\_domain'\] = 'loupvirtuel.fr';

Editez enfin le fichier de configuration apache.conf :

\[srvdmz@srvdmz:~$\] sudo nano apache.conf

et décommentez l'alias situé en début de fichier :

Alias /roundcube /var/lib/roundcube/public\_html

Pour traiter l'alias, redémarrez Apache :

\[srvdmz@srvdmz:~$\] sudo systemctl restart apache2

L'accès à la Bdd roundcube peut être testé avec Adminer, voir [§ 3](/site-web-php-mysql/#3_-_Installation_du_gestionnaire_de_bases_de_donnees_Adminer) [du mémento 8.11](/site-web-php-mysql-debian11/#3_-_Installation_du_gestionnaire_de_Bdd_Adminer) :  
\- Utilisateur > roundcube  
\- Mot de passe > roundcubemysql

#### _12.2 - Utilisation depuis la VM srvlan_

Au préalable, créez sur srvdmz un lien symbolique vers le dossier Web de roundcube :

\[srvdmz@srvdmz:~$\] sudo  ln -s /var/lib/roundcube /var/www/html/roundcube

et donnez à ces 2 dossiers les mêmes permissions que celles du dossier Web de postfixadmin :

\[srvdmz@srvdmz:~$\]  sudo chown -R www-data:www-data /var/lib/roundcube

\[srvdmz@srvdmz:~$\]  sudo chown -R www-data:www-data /var/log/roundcube

Comme avec postfixadmin, la redirection vers HTTPS est assurée depuis cette section située au début du fichier /etc/apache2/sites-available/loupvirtuel.conf :

<VirtualHost \*:80>
ServerName loupvirtuel.fr
ServerAlias www.loupvirtuel.fr srvdmz.loupvirtuel.fr
Redirect permanent / https://loupvirtuel.fr/
</VirtualHost>

Si vous avez exécuté le [mémento 8.21](/https-dotclear-debian11/#5_-_Mise_en_place_du_protocole_HTTPS), les 3 URL HTTP ci-dessous seront redirigées automatiquement vers HTTPS sans faire l'objet d'une alerte de sécurité.

\- http://loupvirtuel.fr/roundcube  
\- http://www.loupvirtuel.fr/roundcube  
\- http://srvdmz.loupvirtuel.fr/roundcube

Renforcez la redirection https en éditant config.inc.php :

\[srvdmz@srvdmz:~$\] cd /etc/roundcube
\[srvdmz@srvdmz:~$\] sudo nano config.inc.php

et en ajoutant les 2 lignes suivantes en fin de fichier :

\# Utilisation forcée du protocole HTTPS
$config\['force\_https'\] = true;

Ensuite, depuis srvlan, lancez l'URL suivante :  
https://loupvirtuel.fr/roundcube

Une fenêtre de connexion Rouncube s'affiche :  
\> Nom d'utilisateur > postmaster@loupvirtuel.fr  
\> Mot de passe > postmaster31  
\> Bouton Connexion

[![Capture - Roundcube : Fenêtre de connexion](/wp-content/uploads/2023/03/roundcube-login-debian11-430x277.jpg "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/roundcube-login-debian11.jpg)

Roundcube : Fenêtre de connexion

Vous devriez trouver dans la boîte de réception affichée par roundcube les courriers transmis à postmaster lors des mémentos précédents.

#### _12.3 - Envoi/réception au sein du LAN_

Depuis debian11-vm1, utilisez roundcube pour envoyer un courrier vers srvlan@loupvirtuel.fr.

Paramètres de connexion sur l'hôte debian11-vm1 :  
\> https://loupvirtuel.fr/roundcube  
\> Nom d'utilisateur > clientmail-vm1@loupvirtuel.fr  
\> Mot de passe > Votre MDP  
\> Bouton Connexion  
\> Menu de gauche, icône Rédiger

Envoyez votre courrier et récupérez celui-ci en utilisant roundcube depuis l'hôte srvlan :

[![Capture - Roundcube : Réception du courrier depuis l'hôte srvlan](/wp-content/uploads/2023/03/roundcube-reception-debian11-430x278.jpg "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/roundcube-reception-debian11.jpg)

Roundcube : Réception du courrier depuis l'hôte srvlan

Réalisez le même test depuis srvlan vers debian11-vm1.

#### _12.4 - Envoi/réception au travers d'Internet_

Référence : [§ 9 du mémento 11.11](/postfix-debian11/#9_-_EnvoiReleve_de-mails_sur_Internet_SPF_DKIM)

Effectuez un envoi depuis le MUA de votre smartphone vers l'adresse x.y@zzz.freeddns.org et récupérez celui-ci depuis l'hôte srvlan et inversement.

Pour vérifier la réception depuis srvlan, utilisez l'URL :  
https://loupvirtuel.fr/roundcube

Une fenêtre de connexion Roundcube s'affiche :  
\> Utilisateur > x.y@zzz.freeddns.org  
\> Mot de passe > y42  
\> Bouton Connexion

#### _12.5 - Ajout des filtres de tri Sieve_

Référence : [§ 2.2 du mémento 11.11](/postfix-debian11/#22_-_Dovecot_MDA_ou_serveur_POPIMAP)

Installez le paquet contenant les plugins de Roundcube :

\[srvdmz@srvdmz:~$\] sudo apt install roundcube-plugins

Ceux-ci ont été installés dans /etc/roundcube/plugins.

Activez le plugin managesieve en éditant ce fichier :

\[srvdmz@srvdmz:~$\] cd /etc/roundcube
\[srvdmz@srvdmz:~$\] sudo nano config.inc.php

et en modifiant la ligne suivante comme ci-dessous :

$config\['plugins'\] = array('managesieve');

Quittez si besoin la session roundcube en cours et reconnectez-vous pour exploiter les filtres.

#### _12.6 - Filtre de réponse auto en cas d'absence_

Créez sur l'hôte debian11-vm2 un filtre permettant de générer une réponse automatique en cas d'absence, ceci pour tous les courriers reçus.

Depuis le navigateur Web, entrez l'URL :  
https://loupvirtuel.fr/roundcube

Une fenêtre de connexion Rouncube s'affiche :  
\> Nom d'utilisateur > clientmail-vm2@loupvirtuel.fr  
\> Mot de passe > Votre MDP  
\> Bouton Connexion

Cliquez sur la roue dentée pour afficher les paramètres :  
\> Volet Paramètres > Filtres  
\> Volet Filtres > Cliquez sur l'icône \+ Créer

Remplissez la définition du filtre comme le montre la capture ci-dessous et enregistrez celui-ci :

[![Capture - Roundcube : Filtre de réponse automatique](/wp-content/uploads/2023/03/roundcube-absence-debian11-430x241.jpg "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/roundcube-absence-debian11.jpg)

Roundcube : Création filtre de réponse automatique

3 représente le nombre de jours pendant lesquels le message d'absence ne sera pas renvoyé à l'expéditeur, peu importe le nombre de courriers qu'il vous envoie.

Testez le filtre depuis le Thunderbird de l'hôte srvlan en envoyant un e-mail de postmaster vers clientmail-vm2.

Pensez ensuite à désactiver le filtre, voir la zone Le filtre est activé visible sur la capture d'écran ci-dessus.

Par curiosité, contrôlez sur le Thunderbird de srvlan qu'un module de nom Sieve peut être installé depuis son menu Modules complémentaires et thèmes.

#### _12.7 - Filtre de transfert automatique_

Créez, toujours depuis clientmail-vm2 connecté, un second filtre permettant le transfert automatique de tous les courriers provenant de postmaster vers srvlan :

[![Capture - Roundcube : Filtre de transfert automatique](/wp-content/uploads/2023/03/roundcube-redirection-debian11-430x220.jpg "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/roundcube-redirection-debian11.jpg)

Roundcube : Création filtre de transfert automatique

Envoyez ensuite depuis le Thunderbird de srvlan un courrier de postmaster vers clientmail-vm2 et vérifiez le transfert de celui-ci sur le compte srvlan@loupvirtuel.fr.

[![Capture -Thunderbird : Courrier pour clientmail-vm2 transféré vers srvlan](/wp-content/uploads/2023/03/thunderbird-redirection-debian11-430x215.jpg "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/thunderbird-redirection-debian11.jpg)

Thunderbird : Courrier pour clientmail-vm2 transféré vers srvlan

Pensez à désactiver le filtre une fois le test terminé.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà, c'est fini. Si administrer à  
distance le réseau virtuel vous tente,  
alors le mémento 12.1 vous attend  
pour découvrir l'outil Cockpit.

[Mémento 12.1](https://familleleloup.no-ip.org/administration-cockpit/)
