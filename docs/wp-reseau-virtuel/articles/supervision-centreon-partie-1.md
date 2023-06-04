---
title: "Centreon / Debian 11 : 1/6"
date: "2023-04-07"
categories: 
  - "outil-centreon"
---

## Mémento 12.2 - Installation de base

### 1 - Installation de Centreon

Centreon pourra superviser les PC de votre réseau local ainsi que les VM de votre réseau virtuel.

Son installation à réaliser de préférence sur l'un des serveurs de votre réseau local disposant de VirtualBox se fera à l'intérieur d'une VM Debian 11.

#### _1.1 - Création de la VM Debian 11_

Aidez-vous du mémento [srvlan – VirtualBox / Debian 11](/serveur-debian11-srvlan-creation/) pour créer la VM avec un bureau Xfce.

Attention, la VM srvlan avait été créée sous VBox 6, c'est légèrement différent sous VBox 7.

Sous VBox 7, panneau Créer une machine virtuelle, cochez Skip Unattended Installation.

Affectez ensuite ces valeurs à la VM lors de sa création :

- Nom > vm-centreon

- Taille de la mémoire > 2048 Mo

- Emplacement du fichier et taille > 12 Go

- Processeur > 2 CPU

- Mode d'accès réseau > Accès par pont _(carte 1)_

Affectez celles-ci lors de l'installation de l'OS Debian :

- Nom de machine > centreon

- Identifiant ... compte utilisateur > user1

- Cochez ... Xfce _(environnement de bureau)_

Une fois Debian 11 installé, la VM rebootera.

Connectez-vous et, comme pour la VM srvlan, autorisez l'usage de sudo à l'utilisateur user1.

Relancez la VM, connectez-vous et ajoutez, comme pour la VM srvlan, les utilitaires de VBox.

Puis, aidez-vous du mémento [Contrôle à distance / Debian 11](/acces-locaux-distants-debian11/#21_-_Installation_dun_serveur_RDP_sur_srvlan) pour installer un serveur xrdp.

Relevez l'IP locale de la VM à l'aide de la Cde ip address, vous en aurez besoin pour configurer votre outil de connexion à distance.

Stoppez à présent la VM et redémarrez-la sous VirtualBox en mode Démarrage sans affichage.

Celle-ci doit maintenant être accessible à distance depuis divers clients RDP :

[![Capture - VM Debian : Exemple d'accès depuis le client RDP mRemoteNG](/wp-content/uploads/2023/03/centreon-base-debian11-430x228.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-base-debian11.webp)

VM Debian : Exemple d'accès depuis le client RDP mRemoteNG

Si tout est OK, vous voilà prêt pour installer Centreon IT.

#### _1.2 - Ajout de Centreon sur la VM_

La procédure présentée ci-dessous s'inspire de la [docs.centreon.com](/wp-content/uploads/2023/03/Centreon-installation-debian-11.pdf).

Mettez à jour la VM Debian 11 depuis son terminal :

sudo apt update  
sudo apt upgrade

Puis, ajoutez les dépendances nécessaires à Centreon :

sudo apt install lsb-release ca-certificates apt-transport-https software-properties-common wget gnupg2

Ajoutez le dépôt de PHP 8.1, Debian 11 ne fournissant à ce jour que PHP 7.4 :

su root 

echo "deb https://packages.sury.org/php/ $(lsb\_release -sc) main" | tee /etc/apt/sources.list.d/sury-php.list 

wget -O- https://packages.sury.org/php/apt.gpg | gpg --dearmor | tee /etc/apt/trusted.gpg.d/php.gpg  > /dev/null 2>&1 

apt update

En revanche, Debian 11 fournit bien MariaDB 10.5.

Ajoutez maintenant le dépôt de Centreon :

echo "deb https://apt.centreon.com/repository/22.10/ $(lsb\_release -sc) main" | tee /etc/apt/sources.list.d/centreon.list

wget -O- https://apt-key.centreon.com | gpg --dearmor | tee /etc/apt/trusted.gpg.d/centreon.gpg > /dev/null 2>&1

apt update

et installez enfin le serveur Centreon :

apt install -y centreon
systemctl daemon-reload
systemctl restart mariadb

Un serveur Apache a été activé lors de l'opération.

Modifiez le fuseau horaire en éditant centreon.ini :

cd /etc/php/8.1/mods-available/ 
nano centreon.ini

et en remplaçant la ligne date.timezone = UTC par date.timezone = Europe/Paris.

Idem pour le fichier /etc/php/8.1/fpm/php.ini, décommentez la ligne date.timezone et affectez lui la même valeur que ci-dessus.

Activez le démarrage automatique des services suivants au boot du système :

systemctl enable php8.1-fpm apache2 centreon cbd centengine gorgoned centreontrapd snmpd snmptrapd

Idem pour le service MariaDB :

systemctl enable mariadb
systemctl restart mariadb

Sécurisez l'accès aux Bdd du serveur MariaDB :

mysql\_secure\_installation

en répondant aux questions posées comme suit :

Enter ... password for root ... : Appuyez sur Entrée  
Switch to unix\_socket authentication \[Y/n\] n

Change the root password? \[Y/n\] y  
New password: Votre MDP root pour MySQL

Remove anonymous users? \[Y/n\] y  
Disallow root login remotely? \[Y/n\] n  
Remove test database and access to it? \[Y/n\] y  
Reload privilege tables now? \[Y/n\] y

Vous êtes maintenant prêt pour l'étape suivante.

#### _1.3 - Ajout de l'interface Web Centreon_

Ouvrez depuis le navigateur Web de la VM Debian 11 l'URL http://localhost :

Puis, suivez cette page de la [docs.centreon.com](/wp-content/uploads/2023/03/Centreon-installation-interface-web.pdf) jusqu'à la partie Initialisation de la supervision.

Deux minutes plus tard, vous devriez pouvoir accéder au tableau de bord de Centreon :

[![Capture - Centreon : Tableau de bord](/wp-content/uploads/2023/03/centreon-accueil-debian11-430x268.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-accueil-debian11.webp)

Centreon : Tableau de bord

Cliquez sur l'icône située en haut et à droite du tableau de bord et sélectionnez Edit profile.

Modifiez la langue à fr\_FR et sauvegardez.

#### _1.4 - Initialisation de la supervision_

Premièrement, entrez les 2 Cdes suivantes dans le terminal de la VM Debian 11 pour éviter de rencontrer des problèmes de permissions :

chown www-data:centreon-engine /etc/centreon-engine/\*
chown www-data:centreon-broker /etc/centreon-broker/\*

Puis, suivez la partie [Initialisation de la supervision](/wp-content/uploads/2023/03/Centreon-installation-interface-web.pdf) du même document que ci-dessus.

Si tout est OK, la page Collecteurs montrera ceci :

[![Capture - Centreon : Page Collecteurs](/wp-content/uploads/2023/03/centreon-collecteur-central-debian11-430x131.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-collecteur-central-debian11.webp)

Centreon : Page Collecteurs

#### _1.5 - Licence solution gratuite Centreon IT-100_

Suivez cette page de la [docs.centreon.com](/wp-content/uploads/2023/03/Centreon-IT100-licence.pdf) pour obtenir votre licence, la démarche est simple.

Vous pourrez ansi, depuis votre serveur central Centreon, superviser jusqu'à 100 hôtes, un hôte étant un PC ou une VM disposant d'une adresse IP/DNS.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Maintenant que Centreon est  
installé, la partie 2 vous attend  
pour découvrir la supervision de  
serveurs NAS Synology et Qnap.

[Partie 2](https://familleleloup.no-ip.org/supervision-centreon-nas-partie-2/)
