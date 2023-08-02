---
title: "srvdmz - VBox/Deb12"
date: "20213-08-02"
categories: 
  - "serveur-srvdmz"
---

## Mémento 3.1 - Serveur srvdmz

IPFire étant installé, vous allez à présent créer le serveur srvdmz que vous placerez en zone dite démilitarisée et qui fournira plus tard des services tels que le Web, FTP, DNS, Mail, etc... pour Internet et le réseau local.

La configuration sera identique à celle de la VM srvlan hormis la partie réseau.

### 1 - Construction de la VM depuis VirtualBox

L'utilisation de VirtualBox est considérée acquise.

A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](../virtualbox-installation/)  
[VirtualBox - Mode d’accès réseau par pont](../virtualbox-pont-reseau/)

#### _1.1 - Création et configuration de la VM_

Le PC hôte doit être un PC 64 bits, courant de nos jours.

Téléchargez l'ISO debian-12.x.y-amd64\-netinst.iso :  
[https://cdimage.debian.org/.../current/amd64/iso-cd/](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)

\- Démarrez ensuite l'application VirtualBox 7.x, puis :  
\- - Menu de VirtualBox > Machine > Nouvelle...  
\-> Nom : srvdmz DMZ  
\-> Folder : Sélectionnez le dossier de stockage des VM  
\-> ISO Image : Sélectionnez l'ISO téléchargée ci-dessus  
\-> Type : Linux  
\-> Version : Debian (64-bit)  
\-> Cochez Skip Unattended Installation _(important)_  
\-> Bouton Suivant

\-> Mémoire vive : 1024 MB  
\-> Processors : 2 CPU si possible  
\-> Bouton Suivant

\-> Create a Virtual Hard Disk Now : Ajustez à 12 Go  
\-> Bouton Suivant

\-> Vérifiez le Récapitulatif  
\-> Bouton Finish

La VM s'affiche dans le panneau gauche de VirtualBox.

\- Sélectionnez maintenant la nouvelle VM, puis :  
\- - Menu de VirtualBox > Machine > Configuration...  
\- - - Onglet Général  
\-> Avancé > Presse-papier partagé > Bidirectionnel

\- - - Onglet Système  
\-> Carte mère > Ordre d'amorçage > Décochez Disquette  
\-> Processeur > Cochez Activer PAE/NX

Facultatif, accès au dossier partagé par le PC hôte :  
\- - - Onglet Dossiers partagés  
\-> Cliquez sur l'icône + _(Ajoute un ... dossier partagé.)_  
\-> Chemin du dossier > Sélectionnez Autre...  
\-> Accédez à votre dossier > Ex : C:\Partage-Windows  
\-> Sélectionner un dossier ou Ouvrir > OK > OK

Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de la distribution Debian_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

\- - Menu de VirtualBox > Machine > Démarrer  
\-> Démarrage normal _(La VM s'exécute)_

Sélectionnez Graphical Install et appliquez ce qui suit :  
\- Language > Français  
\- Pays _(territoire ou région)_ \> France  
\- Disposition de clavier à utiliser > Français  
\- Nom de machine > srvdmz  
\- Domaine > Laissez le champ vide  
\- MDP du super utilisateur root > Votre MDP root  
\- Confirmation du MDP > Votre MDP root  
\- Nom complet du nouvel utilisateur > Ex: srvdmz  
\- Identifiant pour le compte utilisateur > srvdmz  
\- MDP pour le nouvel utilisateur > Votre MDP srvdmz  
\- Confirmation du MDP > Votre MDP srvdmz  
\- Méthode de partitionnement > Assisté - utili... entier  
\- Disque à partitionner > Celui proposé de 12 Go  
\- Schéma de partitionnement > Tout ... seule partition  
\- Table des partitions > Terminer le partitionnement ..  
\- Faut-il appliquer les changements ... disques ? > Oui

L'installation commence :  
\- Faut-il analyser d'autres supports ... ? > Non  
\- Pays du miroir de l'archive Debian > France  
\- Miroir de l'archive Debian > deb.debian.org  
\- Mandataire HTTP (lais...) > Laissez vide

L'installation continue :  
\- Souhaitez-vous participer à l'étude statistique ... > Non  
\- Logiciels à installer  
\-> Décochez environnement de bureau Debian  
\-> Décochez ... GNOME  
\-> Cochez ... Xfce  
\-> Conservez utilitaires usuels du système

L'installation se termine :  
\- Installer ... de démarrage GRUB sur le disque ... > Oui  
\- Périphérique ... programme de démarrage > /dev/sda  
\- Installation terminée > Continuer _(sans retrait du CD)_

Le système reboot et une fenêtre de connexion s'ouvre :

[![Capture - Fenêtre de connexion Xfce](/wp-content/uploads/2021/09/srvdmz-deb11-login-430x370.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/09/srvdmz-deb11-login.jpg)

Fenêtre de connexion Xfce

\- Premier champ : Entrez srvdmz  
\- Second champ : Entrez Votre MDP srvdmz  
\> Bouton Se connecter

Le bureau Xfce s'ouvre :

[![Capture - bureau Xfce](/wp-content/uploads/2021/09/srvdmz-deb11-bureau1-xfce-430x370.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/09/srvdmz-deb11-bureau1-xfce.jpg)

Bureau Xfce

Ouvrez le terminal de Cdes en cliquant sur son icône située en bas à l'intérieur du dock.

Autorisez l'usage de sudo à l'utilisateur srvdmz :

\[srvdmz@srvdmz:~$\] su root
Mot de passe : Votre MDP root
\[root@srvdmz:~#\] sudo usermod -aG sudo srvdmz
\[root@srvdmz:~#\] exit

et redémarrez le serveur :  
\> Menu Applications de Xfce situé en haut à gauche  
\> Déconnexion > Une fenêtre s'ouvre  
\> Bouton Redémarrer

Reconnectez-vous ensuite en tant qu'utilisateur srvdmz et rouvrez le terminal de Cdes.

### 2 - Installation des utilitaires de VirtualBox

Ils permettront entre autres le copier/coller et l'accès au dossier partagé par le PC hôte.

Notez au préalable la version courante du noyau linux :

\[srvdmz@srvdmz:~$\] uname -r

Retour = 5.10.0-x\-amd64

Installez ensuite les 2 paquets suivants :

\[srvdmz@srvdmz:~$\] sudo apt install dkms build-essential

et si nécessaire le paquet linux-headers approprié :

\[srvdmz@srvdmz:~$\] sudo apt install linux-headers-5.10.0-x\-amd64

puis accédez au menu VirtualBox de la fenêtre VM :  
\> Périphériques > Insérer l'image CD des Additions inv...

Montez l'image CD, installez les utilitaires et rebootez :

\[srvdmz@srvdmz:~$\] sudo mount /dev/cdrom /media/cdrom
\[srvdmz@srvdmz:~$\] cd /media/cdrom
\[srvdmz@srvdmz:~$\] sudo ./VBoxLinuxAdditions.run
\[srvdmz@srvdmz:~$\] sudo reboot

Une fois fini, reconnectez-vous, la fenêtre VM srvdmz peut à présent être redimensionnée avec la souris. Sa nouvelle taille sera enregistrée au sein de la VM.

Le copier/coller entre le PC hôte et srvdmz doit maintenant fonctionner dans les 2 sens.

Sans fermer la VM, retirez l'image CD du lecteur virtuel :  
Menu Configuration de VirtualBox :  
\- - - Onglet Stockage  
\> Zone Unités de stockage > VBoxGuestAdditions.iso  
\> Zone Attributs > Cliquez sur l'icône CD  
\> Retirer le disque du lecteur virtuel \> OK

### 3 - Suppression d'applications préinstallées, etc...

Supprimez les applications non utiles sur srvdmz :

\[srvdmz@srvdmz:~$\] sudo apt autoremove --purge \\
libreoffice-writer libreoffice-impress \\
libreoffice-calc libreoffice-math \\
libreoffice-draw libreoffice-base-core \\
libreoffice-core libreoffice-common

\[srvdmz@srvdmz:~$\] sudo rm -r /etc/libreoffice
\[srvdmz@srvdmz:~$\] sudo rm -r /usr/share/fonts/truetype/libreoffice

\[srvdmz@srvdmz:~$\] sudo apt autoremove --purge \\
exfalso quodlibet

et mettez le navigateur firefox en français :

\[srvdmz@srvdmz:~$\] sudo apt install firefox-esr-l10n-fr

La configuration de base est presque terminé :

[![Capture - Debian 11 : Bureau Xfce personnalisé](/wp-content/uploads/2021/09/srvdmz-deb11-bureau-xfce-bis-430x288.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/09/srvdmz-deb11-bureau-xfce-bis.jpg)

Debian 11 : Bureau Xfce personnalisé

La consommation RAM actuelle est d'environ 500 Mo.

### 4 - Montage du dossier partagé par le PC hôte

Créez le dossier qui permettra d'afficher le contenu partagé par le PC hôte :

\[srvdmz@srvdmz:~$\] mkdir /home/srvdmz/Partage

Créez le fichier home-srvdmz-Partage.mount :

\[srvdmz@srvdmz:~$\] cd /etc/systemd/system
\[srvdmz@srvdmz:~$\] sudo touch home-srvdmz-Partage.mount

Editez celui-ci :

\[srvdmz@srvdmz:~$\] sudo nano home-srvdmz-Partage.mount

et entrez le contenu suivant :

\[Unit\]
Description = Montage dossier partagé fourni par VirtualBox

\[Mount\]
What = _Entrez le nom du dossier partagé par le PC hôte_
Where = /home/srvdmz/Partage
Type = vboxsf
Options=rw,uid=srvdmz,gid=srvdmz

\[Install\]
WantedBy = multi-user.target

Exemple pour What : What=Partage-Alfred

Intégrez le service dans la configuration de systemd :

\[srvdmz@srvdmz:~$\] sudo systemctl daemon-reload

et démarrez celui-ci :

\[srvdmz@srvdmz:~$\] sudo systemctl start home-srvdmz-Partage.mount

Vérifiez son statut :

\[srvdmz@srvdmz:~$\] sudo systemctl status home-srvdmz-Partage.mount

Si nécessaire, touche q pour quitter le résultat affiché.

Autorisez le lancement du service au boot de la VM :

\[srvdmz@srvdmz:~$\] sudo systemctl enable home-srvdmz-Partage.mount

Un lien symbolique est normalement créé.

Ouvrez le gestionnaire de fichiers thunar et observez le contenu de /home/srvdmz/Partage.

### 5 - Configuration du réseau

Avant, vérifiez les IP courantes avec la Cde ip address :

\[srvdmz@srvdmz:~$\] ip address

Résultat :

![Capture - Résultat de la Cde ip address](/wp-content/uploads/2021/09/srvdmz-deb11-ip-address.jpg)

Résultat de la Cde ip address

La VM, prévue en zone DMZ, exige de changer le mode d'accès réseau de la carte enp0s3.

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
\> Champ Adresse : Entrez 192.168.4.2  
\> Champ Masque de réseau : Entrez 255.255.255.0  
\> Champ Passerelle : Entrez 192.168.4.1  
\> Serveurs DNS > Entrez l'IP locale de votre Box Internet  
\> Bouton Enregistrer  
  
Fermez ensuite la fenêtre Connexions réseau.  
Le service réseau redémarre automatiquement.

Vérifiez par prudence la bonne configuration du réseau :

\[srvdmz@srvdmz:~$\] ip address
\[srvdmz@srvdmz:~$\] nmcli           \# Cde NetworkManager

![Capture - Résultat de la Cde nmcli](/wp-content/uploads/2021/09/srvdmz-deb11-cde-nmcli.jpg)

Résultat de la Cde nmcli

Les fichiers configurés avec networkmanager sont ici :  
/etc/NetworkManager/system-connections/

_Nota : La VM srvdmz accède à Internet si la VM srvsec est démarrée._

Le service réseau peut être redémarré avec cette Cde :

\[srvdmz@srvdmz:~$\] sudo systemctl restart NetworkManager

Par curiosité, lisez la table de routage avec la Cde ip r :

![Capture - Table de routage courante de srvdmz ](/wp-content/uploads/2021/09/srvdmz-deb11-table-routage.jpg)

Table de routage courante de srvdmz

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Bien !  
Les serveurs sont prêts. Le mémento  
4.11 vous attend pour la création des  
VM clientes du réseau local virtuel.

[Mémento 4.11](https://familleleloup.no-ip.org/virtualbox-clients-debian11-creation/)
