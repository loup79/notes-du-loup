---
title: "srvlan - VBox/Deb11"
date: "2021-08-27"
categories: 
  - "serveur-srvlan"
---

## Mémento 1.11 - Serveur srvlan

L'architecture du réseau virtuel proposée sous Debian 11 sera la même que sous Debian 10 mais le serveur srvlan supportera cette fois l'environnement graphique Xfce.

Ce dernier, allégé de quelques applications préinstallées, contribuera au confort d'exploitation du serveur.

Par souci d'homogénéité, Xfce sera installé sur les serveurs et clients graphiques du réseau.

### 1 - Construction de la VM depuis VirtualBox

L'utilisation de VirtualBox est considérée acquise.

A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](/virtualbox-installation/)  
[VirtualBox - Mode d’accès réseau par pont](/virtualbox-pont-reseau/)

#### _1.1 - Création et configuration_

Le PC hôte doit être un PC 64 bits, courant de nos jours.

Téléchargez l'ISO debian-11.x.y-amd64\-netinst.iso :  
[https://cdimage.debian.org/.../current/amd64/iso-cd/](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)

Démarrez ensuite l'application VirtualBox, puis :

Menu Nouvelle de VirtualBox :  
\- Nom srvlan LAN - Type Linux - Version Debian (64-bit)  
\- Taille de la mémoire > 1024 Mo  
\- Disque dur > Créer un disque dur virtuel maintenant  
\- Type de fichier de disque dur > VDI  
\- Stockage sur disque dur ... > Dynamiquement alloué  
\- Emplacement du fichier et taille > 12 Go > Créer

La VM est créée dans le panneau gauche de VirtualBox.

Sélectionnez la nouvelle VM, puis :

Menu Configuration de VirtualBox :  
\- - - Onglet Général  
\> Avancé > Presse-papier partagé > Bidirectionnel  
  
\- - - Onglet Système  
\> Carte mère > Ordre d'amorçage > Décochez Disquette  
\> Carte mère > Fonctions avancées > Cochez IO-APIC  
\> Processeur > 2 CPU et cochez PAE/NX  
  
\- - - Onglet Affichage  
\> Ecran > Contrôleur graphique > VMSVGA  
  
\- - - Onglet Stockage  
\> Zone Unités de stockage > Sélectionnez Vide  
\> Zone Attributs > Cliquez sur l'icône CD  
\> Sélectionnez Choisissez un fichier de disque ...  
\> Entrez le chemin de l'image ISO Debian > Ouvrir  
  
\- - - Onglet Réseau  
\> Carte 2 > Cochez Activer la carte réseau  
\> Mode d'accès réseau > Sélectionnez Réseau interne

Facultatif, accès au dossier partagé par le PC hôte :  
\- - - Onglet Dossiers partagés  
\> Cliquez sur l'icône + > Ajouter un dossier partagé  
\> Chemin du dossier > Sélectionnez Autre...  
\> Accédez à votre dossier > Ex : C:\\Partage-Windows  
\> Sélectionner un dossier ou Ouvrir > OK > OK

Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de la distribution Debian_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

Menu Démarrer de VirtualBox :  
La VM créée s'exécute.

Sélectionnez Graphical Install et appliquez ce qui suit :  
\- Language > Français  
\- Pays _(territoire ou région)_ \> France  
\- Disposition de clavier à utiliser > Français  
\- Interface réseau principale > enp0s3  
\- Nom de machine > srvlan  
\- Domaine > Laissez le champ vide  
\- MDP du super utilisateur root > Votre MDP root  
\- Confirmation du MDP > Votre MDP root  
\- Nom complet du nouvel utilisateur > Ex: srvlan  
\- Identifiant pour le compte utilisateur > srvlan  
\- MDP pour le nouvel utilisateur > Votre MDP srvlan  
\- Confirmation du MDP > Votre MDP srvlan  
\- Méthode de partitionnement > Assisté - utili... entier  
\- Disque à partitionner > Celui proposé de 12 Go  
\- Schéma de partitionnement > Tout ... seule partition  
\- Table des partitions > Terminer le partitionnement ..  
\- Faut-il appliquer les changements ... disques ? > Oui

L'installation de base commence :  
\- Faut-il analyser d'autres supports ... ? > Non  
\- Pays du miroir de l'archive Debian > France  
\- Miroir de l'archive Debian > deb.debian.org  
\- Mandataire HTTP (lais...) > Laissez vide

L'installation continue :  
\- Souhaitez-vous participer à l'étude statistique ... > Non  
\- Logiciels à installer  
\> Décochez environnement de bureau Debian  
\> Décochez ... GNOME  
\> Cochez ... Xfce  
\> Conservez utilitaires usuels du système

L'installation se termine :  
\- Installer ... de démarrage GRUB sur le disque ... > Oui  
\- Périphérique ... programme de démarrage > /dev/sda  
\- Installation terminée > Continuer _(sans retrait du CD)_

Le système reboot et une fenêtre de connexion s'ouvre :

[![Capture - Fenêtre de connexion Xfce](/wp-content/uploads/2021/08/srvlan-deb11-login-430x372.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/08/srvlan-deb11-login.jpg)

Fenêtre de connexion Xfce

\- Premier champ : Entrez srvlan  
\- Second champ : Entrez Votre MDP srvlan  
\> Bouton Se connecter

Le bureau Xfce s'ouvre :

[![Capture - bureau Xfce](/wp-content/uploads/2021/08/srvlan-deb11-bureau1-xfce-430x368.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/08/srvlan-deb11-bureau1-xfce.jpg)

Bureau Xfce

Ouvrez le terminal de Cdes en cliquant sur son icône située en bas à l'intérieur du dock.

Autorisez l'usage de sudo à l'utilisateur srvlan :

\[srvlan@srvlan:~$\] su root
Mot de passe : _Votre MDP root_
\[root@srvlan:~#\] sudo usermod -aG sudo srvlan
\[root@srvlan:~#\] exit

et redémarrez le serveur :  
\> Menu Applications de Xfce situé en haut à gauche  
\> Déconnexion > Une fenêtre s'ouvre  
\> Bouton Redémarrer

Reconnectez-vous ensuite en tant qu'utilisateur srvlan et rouvrez le terminal de Cdes.

### 2 - Installation des utilitaires de VirtualBox

Ils permettront entre autres le copier/coller et l'accès au dossier partagé par le PC hôte.

Notez au préalable la version courante du noyau linux :

\[srvlan@srvlan:~$\] uname -r

Retour = 5.10.0-x\-amd64

Installez ensuite les 2 paquets suivants :

\[srvlan@srvlan:~$\] sudo apt install dkms build-essential

et si nécessaire le paquet linux-headers approprié :

\[srvlan@srvlan:~$\] sudo apt install linux-headers-5.10.0-x\-amd64

puis accédez au menu VirtualBox de la fenêtre VM :  
\> Périphériques > Insérer l'image CD des Additions inv...

Montez l'image CD, installez les utilitaires et rebootez :

\[srvlan@srvlan:~$\] sudo mount /dev/cdrom /media/cdrom
\[srvlan@srvlan:~$\] cd /media/cdrom
\[srvlan@srvlan:~$\] sudo ./VBoxLinuxAdditions.run
\[srvlan@srvlan:~$\] sudo reboot

Une fois fini, reconnectez-vous, la fenêtre VM srvlan peut à présent être redimensionnée avec la souris. Sa nouvelle taille sera enregistrée au sein de la VM.

Le pratique copier/coller entre le PC hôte et srvlan doit maintenant fonctionner dans les 2 sens.

Sans fermer la VM, retirez l'image CD du lecteur virtuel :  
Menu Configuration de VirtualBox :  
\- - - Onglet Stockage  
\> Zone Unités de stockage > VBoxGuestAdditions.iso  
\> Zone Attributs > Cliquez sur l'icône CD  
\> Retirer le disque du lecteur virtuel > OK

### 3 - Suppression d'applications préinstallées, etc...

Supprimez les applications non utiles sur srvlan :

\[srvlan@srvlan:~$\] sudo apt autoremove --purge \\
libreoffice-writer libreoffice-impress \\
libreoffice-calc libreoffice-math \\
libreoffice-draw libreoffice-base-core \\
libreoffice-core libreoffice-common

\[srvlan@srvlan:~$\] sudo rm -r /etc/libreoffice
\[srvlan@srvlan:~$\] sudo rm -r /usr/share/fonts/truetype/libreoffice

\[srvlan@srvlan:~$\] sudo apt autoremove --purge \\
exfalso quodlibet

et mettez le navigateur firefox en français :

\[srvlan@srvlan:~$\] sudo apt install firefox-esr-l10n-fr

La configuration de base est presque terminé :

[![Capture - Debian 11 : Bureau Xfce personnalisé](/wp-content/uploads/2021/08/srvlan-deb11-bureau-xfce-bis-430x288.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/08/srvlan-deb11-bureau-xfce-bis.jpg)

Debian 11 : Bureau Xfce personnalisé

La consommation RAM actuelle est d'environ 500 Mo.

### 4 - Montage du dossier partagé par le PC hôte

Créez le dossier qui permettra d'afficher le contenu partagé par le PC hôte :

\[srvlan@srvlan:~$\] mkdir /home/srvlan/Partage

Créez le fichier de service home-srvlan-Partage.mount :

\[srvlan@srvlan:~$\] cd /etc/systemd/system
\[srvlan@srvlan:~$\] sudo touch home-srvlan-Partage.mount

Editez celui-ci :

\[srvlan@srvlan:~$\] sudo nano home-srvlan-Partage.mount

et entrez le contenu suivant :

\[Unit\]
Description = Montage dossier partagé fourni par VirtualBox

\[Mount\]
What = _Entrez le nom du dossier partagé par le PC hôte_
Where = /home/srvlan/Partage
Type = vboxsf
Options=rw,uid=srvlan,gid=srvlan

\[Install\]
WantedBy = multi-user.target

Exemple pour What : What=Partage-Alfred

Intégrez le service dans la configuration de systemd :

\[srvlan@srvlan:~$\] sudo systemctl daemon-reload

et démarrez celui-ci :

\[srvlan@srvlan:~$\] sudo systemctl start home-srvlan-Partage.mount

Vérifiez son statut :

\[srvlan@srvlan:~$\] sudo systemctl status home-srvlan-Partage.mount

Si nécessaire, touche q pour quitter le résultat affiché.

Autorisez le lancement du service au boot de la VM :

\[srvlan@srvlan:~$\] sudo systemctl enable home-srvlan-Partage.mount

Un lien symbolique est normalement créé.

Ouvrez le gestionnaire de fichiers thunar et observez le contenu de /home/srvlan/Partage.

### 5 - Configuration du réseau

Avant, vérifiez les IP courantes avec la Cde ip address :

\[srvlan@srvlan:~$\] ip address

Résultat :

![Capture - Résultat de la Cde ip address](/wp-content/uploads/2021/08/srvlan-deb11-ip-address.jpg)

Résultat de la Cde ip address

Puis installer les 2 paquets suivants :

\[srvlan@srvlan:~$\] sudo apt install netfilter-persistent

\[srvlan@srvlan:~$\] sudo apt install iptables-persistent

Une fenêtre Config... iptables-persistent s'ouvre :  
\> Faut-il enregistrer les règles IPv4 actuelles ? > Oui  
\> Faut-il enregistrer les règles IPv6 actuelles ? > Oui

2 fichiers rules.v4/v6 ont été créés dans /etc/iptables/.  
Ils serviront à déclarer persistantes des règles iptables.

La VM srvlan, prévue en zone LAN, exige de changer le mode d'accès réseau de la carte enp0s3.

Pour cela, allez au menu Configuration de VirtualBox :  
\- - - Onglet Réseau  
\> Carte 1 > Mode d'accès réseau > Réseau interne  
\> OK

#### _5.1 - Adressage IP fixe carte enp0s3_

Configurez à présent une IP fixe sur la carte enp0s3 :  
\- - Bureau Xfce, barre du haut  
\> Clic droit sur l'icône Réseau située à droite  
\> Sélectionnez Modifier les connexions...  
  
Une fenêtre Connexions réseau s'ouvre :  
\> Sélectionnez la connexion Wired connection 1  
\> Cliquez sur l'icône roue dentée située en bas à gauche  
  
Une fenêtre Modification de ... s'ouvre :  
\> Nom de la connexion > Connexion Ethernet 1  
  
\- - - Onglet Ethernet  
\> Périphérique > Sélectionnez enp0s3  
  
\- - - Onglet Paramètres IPv4  
\> Méthode > Sélectionnez Manuel > Bouton Ajouter  
\> Champ Adresse : Entrez 192.168.2.2  
\> Champ Masque de réseau : Entrez 255.255.255.0  
\> Champ Passerelle : Entrez 192.168.2.1  
\> Serveurs DNS > Entrez l'IP locale de votre Box Internet  
\> Bouton Enregistrer  
  
Fermez ensuite la fenêtre Connexions réseau.  
Le service réseau redémarre automatiquement.

Pour finir, vérifiez le résultat avec la Cde ip address.

#### _5.2 - Adressage IP fixe carte enp0s8_

Configurez également une IP fixe sur la carte enp0s8 :  
\- - Bureau Xfce, barre du haut  
\> Clic droit sur l'icône Réseau située à droite  
\> Sélectionnez Modifier les connexions...  
  
La fenêtre Connexions réseau s'ouvre :  
\> Cliquez sur l'icône + située en bas à gauche  
  
Une fenêtre Sélectionner ... de connexion s'ouvre :  
\> Sélectionnez Ethernet > Bouton Créer...  
  
Une fenêtre Modification de ... s'ouvre :  
\> Nom de la connexion > Connexion Ethernet 2  
  
\- - - Onglet Ethernet  
\> Périphérique > Sélectionnez enp0s8  
  
\- - - Onglet Paramètres IPv4  
\> Méthode > Sélectionnez Manuel > Bouton Ajouter  
\> Champ Adresse : Entrez 192.168.3.1  
\> Champ Masque de réseau : Entrez 255.255.255.0  
\> Bouton Enregistrer  
  
Fermez ensuite la fenêtre Connexions réseau.  
Le service réseau redémarre automatiquement.

Vous venez de créer 2 adresses IP fixes.

Vérifiez par prudence la bonne configuration du réseau :

\[srvlan@srvlan:~$\] ip address
\[srvlan@srvlan:~$\] nmcli  \# Cde NetworkManager

Les fichiers configurés avec NetworkManager sont ici :  
/etc/NetworkManager/system-connections/

_Nota : La VM srvlan n'accède plus à Internet, la VM suivante permettra de retrouver cet accès._

#### _5.3 - Routage interne au serveur_

L'activation du routage avec la Cde ip\_forward permet, selon les règles définies dans la table de routage du serveur, le renvoi de paquets de données arrivés par une interface réseau vers une autre interface réseau.

Pour l'activez, éditez le fichier sysctl.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/sysctl.conf

et retirez le # de la ligne #net.ipv4.ip\_forward=1.

Relancez ensuite le service réseau :

\[srvlan@srvlan:~$\] sudo systemctl restart NetworkManager

#### _5.4 - Translation d'adresses NAT_

\-- Définition de WIKIBOOKS -- 
Objectif du NAT _(Network Address Translation)_ :  
Faire que les PC d'un réseau interne n'apparaissent que sous l'identifiant d'une seule IP pour les réseaux externes _(c'est un masquage ou IP Masquerading)_.

Votre box Internet _(passerelle)_ fait du NAT entre votre réseau privé et le réseau Internet et de ce fait votre fournisseur d'accès _(FAI)_ ne vous donne qu'une seule IP alors que vous pouvez très bien avoir plusieurs PC connectés à Internet à partir de votre réseau local.

Le NAT répond principalement au manque d'adresses IP dans le plan d'adressage ipV4 pour l'accès à Internet.

Il permet également de s'affranchir de la gestion des tables de routage et fonctionne avec le service iptables installé par défaut avec Debian.

Vérifiez l'état courant du NAT ou IP Masquerading :

\[srvlan@srvlan:~$\] sudo iptables -L -t nat

Résultat, le NAT est vu comme inactif :

![Capture - Le NAT (translation d'adresses) est vu non activé](/wp-content/uploads/2021/08/srvlan-deb11-iptables-inactif.jpg)

Le NAT (translation d'adresses) est vu non activé

Activez celui-ci en utilisant une règle iptables :

\[srvlan@srvlan:~$\] sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

et affichez de nouveau le contenu de la table NAT :

\[srvlan@srvlan:~$\] sudo iptables -L -t nat

Résultat, le NAT est actif :

![Capture - Le NAT (translation d'adresses) est vu activé](/wp-content/uploads/2021/08/srvlan-deb11-iptables-nat-actif.jpg)

Le NAT (translation d'adresses) est vu activé

Effectuez une sauvegarde de la règle iptables :

\[srvlan@srvlan:~$\] su root
\[root@srvlan:~$\] sudo iptables-save > /etc/iptables/rules.v4
\[root@srvlan:~$\] exit

et déclarez celle-ci persistente :

\[srvlan@srvlan:~$\] sudo systemctl enable netfilter-persistent
\[srvlan@srvlan:~$\] sudo systemctl restart netfilter-persistent

Elle sera ainsi activée à chaque boot du système.

Par curiosité, lisez la table de routage avec la Cde ip r :

![Capture - Table de routage courante de srvlan ](/wp-content/uploads/2021/09/srvlan-deb11-table-routage.jpg)

Table de routage courante de srvlan

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Bravo !  
Le serveur srvlan est prêt.  
Le mémento 2.1 vous attend pour  
l'ajout du serveur srvsec_(IPFire)_.

[Mémento 2.1](https://familleleloup.no-ip.org/serveur-ipfire-srvsec-creation/)
