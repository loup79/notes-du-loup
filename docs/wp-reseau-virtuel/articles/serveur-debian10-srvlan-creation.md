---
title: "srvlan – VirtualBox / Debian 10"
date: "2022-04-30"
categories: 
  - "archives"
---

## Mémento 1.1 - Serveur srvlan

Comme pour le réseau virtuel 2017, srvlan sera la première VM que vous allez construire.

L'installation de l'OS Debian 10 sur la VM sera minimaliste et complétée par la mise en place d'un gestionnaire de fenêtres très léger appelé Openbox.

Ce dernier contribuera au confort d'usage du serveur.

### 1 - Construction de la VM depuis VirtualBox

L'utilisation de VirtualBox est considérée acquise.

A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](/virtualbox-installation/)  
[VirtualBox - Mode d’accès réseau par pont](/virtualbox-pont-reseau/)

#### _1.1 - Création et configuration_

Téléchargez l'ISO 32 bits debian-10.x.y-i386-netinst.iso :  
[https://cdimage.debian.org/.../10.10.0/i386/iso-cd/](https://cdimage.debian.org/cdimage/archive/10.10.0/i386/iso-cd/)

Démarrez ensuite l'application VirtualBox, puis :

Menu Nouvelle de VirtualBox :  
\- Nom srvlan LAN - Type Linux - Version Debian (32-bit)  
\- Taille de la mémoire > 512 Mo  
\- Disque dur > Créer un disque dur virtuel maintenant  
\- Type de fichier de disque dur > VDI  
\- Stockage sur disque dur ... > Dynamiquement alloué  
\- Emplacement du fichier et taille > 12 Go > Créer

La VM est créée dans le panneau gauche de VirtualBox.

Sélectionnez la nouvelle VM, puis :

Menu Configuration :  
\- - - Onglet Général  
\> Avancé > Presse-papier partagé > Bidirectionnel  
  
\- - - Onglet Système  
\> Carte mère > Ordre d'amorçage > Décochez Disquette  
\> Carte mère > Fonctions avancées > Cochez IO-APIC  
\> Processeur > 2 CPU et cochez Activer PAE/NX  
  
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
\> Sélectionner un dossier > OK > OK

Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de la distribution Debian_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

Menu Démarrer de VirtualBox :  
La VM s'exécute, contrôlez l'ISO affichée et démarrez.

Sélectionnez Graphical Install et appliquez ce qui suit :  
\- Language > French  
\- Pays (territoire ou région) > France  
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
\- Faut-il analyser un autre CD ou DVD ? > Non  
\- Pays du miroir de l'archive Debian > France  
\- Miroir de l'archive Debian > ftp.fr.debian.org  
\- Mandataire HTTP (lais...) > Laissez vide

L'installation continue :  
\- Souhaitez-vous participer à l'étude statistique ... > Non  
\- Logiciels à installer  
\> Décochez environnement de bureau Debian  
\> Décochez serveur d'impression  
\> Conservez utilitaires usuels du système

L'installation se termine :  
\- Installer ... de démarrage GRUB sur le secteur ... > Oui  
\- Périphérique ... programme de démarrage > /dev/sda  
\- Installation terminée > Continuer _(sans retrait du CD)_

Le système reboot :  
\- srvlan login : srvlan  
\- Password : Votre MDP srvlan  
Si tout est OK, affichage du prompt \[srvlan@srvlan:~$\]

Connectez-vous à présent comme administrateur root :

\[srvlan@srvlan:~$\] su root  
Mot de passe : Votre MDP root  
\[root@srvlan:~#\]

#### _1.3 - Gestion des paquets Debian_

Dans le but de garder un Debian minimaliste, empêchez les paquets recommandés/suggérés par le gestionnaire de paquets apt d'être installés par défaut comme des dépendances.

Créez et éditez le fichier 00pas-de-recommends :

\[root@srvlan:~#\] cd /etc/apt
\[root@srvlan:~#\] touch apt.conf.d/00pas-de-recommends
\[root@srvlan:~#\] nano apt.conf.d/00pas-de-recommends

Entrez ensuite ces 2 lignes et enregistrez le fichier :

APT::Install-Recommends "false";
APT::Install-Suggests "false";

Touches CTRL + x pour enregistrer > O = Oui > Entrée

#### _1.4 - Installation de sudo et MAJ de Debian_

Autorisez l'utilisateur srvlan à exécuter des Cdes en tant que root à l'aide du programme sudo. Vous diminuerez ainsi, connecté en tant que srvlan, le risque de casser la VM par erreur.

Installez sudo et ajoutez srvlan dans le groupe sudo :

\[root@srvlan:~#\] apt-get install sudo  
\[root@srvlan:~#\] sudo adduser srvlan sudo 

Redémarrez le système pour une prise en compte :

\[root@srvlan:~#\] sudo reboot

et connectez-vous en tant qu'utilisateur srvlan :

[![Capture - Premier démarrage de Debian](/wp-content/uploads/2019/04/srvlan-premier-demarrage-430x370.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/04/srvlan-premier-demarrage.png)

Premier démarrage de Debian

Mettez à jour la distribution Debian :

\[srvlan@srvlan:~$\] sudo apt-get update  
\[srvlan@srvlan:~$\] sudo apt-get upgrade  
\[srvlan@srvlan:~$\] sudo apt-get dist-upgrade

Pour info, la VM peut être arrêtée de 2 façons :

\[srvlan@srvlan:~$\] sudo poweroff  
\[srvlan@srvlan:~$\] sudo shutdown -h now 

Stoppez donc celle-ci, réduisez depuis VirtualBox la mémoire allouée à 384 Mo et redémarrez.

### 2 - Ajout du serveur graphique Xorg

Vous devez, comme préalable à la mise en place d'Openbox, installer un serveur graphique.

Installez xorg à l'aide des Cdes suivantes :

\[srvlan@srvlan:~$\] sudo apt-get install xfonts-base  
\[srvlan@srvlan:~$\] sudo apt-get install xserver-xorg  
\[srvlan@srvlan:~$\] sudo apt-get install x11-xserver-utils  

### 3 - Ajout du gestionnaire de fenêtres Openbox

#### _3.1 - Installation de base_

Entrez la Cde suivante :

\[srvlan@srvlan:~$\] sudo apt-get install openbox obmenu obconf-qt obconf-qt-l10n

#### _3.2 - Gestionnaire de session, terminal ..._

Ajoutez ces 3 programmes avec les Cdes suivantes :

\[srvlan@srvlan:~$\] sudo apt-get install slim xfce4-terminal    
\[srvlan@srvlan:~$\] sudo apt-get install chromium chromium-l10n chromium-sandbox 

Redémarrez le système :

\[srvlan@srvlan:~$\] sudo reboot

Le gestionnaire de session/connexion slim s'ouvre.

Entrez le nom d'utilisateur srvlan > Touche Entrée.  
Entrez ensuite son MDP > Touche Entrée.

Le bureau s'affiche, effectuez un clic droit sur celui-ci pour activer le menu contextuel d'Openbox et vérifiez le fonctionnement de Terminal emulator et Web browser.

Info sur le thème actif du gestionnaire de session slim :  
\- Choix dans /etc/slim.conf, paramètre current\_theme.  
\- Thèmes disponibles dans /usr/share/slim/themes.

#### _3.3 - Barre des tâches_

Ouvrez Terminal emulator et installez la barre tint2 :

\[srvlan@srvlan:~$\] sudo apt-get install tint2

Créez un dossier openbox dans /home/srvlan/.config/ :

\[srvlan@srvlan:~$\] mkdir /home/srvlan/.config/openbox

Créez dans le dossier un fichier autostart.sh :

\[srvlan@srvlan:~$\] cd /home/srvlan/.config/openbox
\[srvlan@srvlan:~$\] touch autostart.sh

Editez celui-ci :

\[srvlan@srvlan:~$\] nano autostart.sh

et insérez les lignes suivantes avant d'enregistrer :

\# Lancement de la barre des tâches  
tint2 &

Redémarrez et découvrez la barre des tâches tint2 :

\[srvlan@srvlan:~$\] sudo reboot

#### _3.4 - Assistant du gestionnaire de session_

Ouvrez le terminal et installez l'outil obsession :

\[srvlan@srvlan:~$\] sudo apt-get install obsession

Editez le menu contextuel utilisé par Openbox :

\[srvlan@srvlan:~$\] sudo nano /etc/xdg/openbox/menu.xml

et remplacez le contenu suivant situé à la fin du fichier :

<item label="Exit">    
   <action name="Exit" />  
</item>

par :

<item label="Exit">  
   <action name="Execute">  
   <execute>obsession-logout</execute>  
   </action>  
</item>

Ouvrez ensuite ce menu _(clic droit sur le bureau)_  
\> Exit pour observer le menu fourni de base  
\> Annuler

\- Ouvrez de nouveau le menu contextuel  
\> Restart pour traiter la modification ci-dessus

\- Ouvrez une dernière fois le menu contextuel  
\> Exit pour voir le nouveau menu fourni par obsession  
\> Annuler

Des boutons Arrêter, Redémarrer, etc... compléteront le menu une fois le paquet policykit-1-gnome installé.

#### _3.5 - Utilitaires pratiques_

Installez les 7 utilitaires suivants :  
\- Le gestionnaire de paquets synaptic  
\- L'éditeur de textes léger mousepad  
\- Le gestionnaire de fichiers pcmanfm  
\- Le créateur de fond d'écran hsetroot  
\- L'agent d'authentification policykit-1-gnome  
\- Le moniteur système conky  
\- L'éditeur d'images gpicview

\[srvlan@srvlan:~$\] sudo apt-get install synaptic mousepad
\[srvlan@srvlan:~$\] sudo apt-get install pcmanfm hsetroot
\[srvlan@srvlan:~$\] sudo apt-get install policykit-1-gnome
\[srvlan@srvlan:~$\] sudo apt-get install conky gpicview

Prévoyez d'en exécuter 3 au démarrage du système.

Editez pour cela le fichier autostart.sh :

\[srvlan@srvlan:~$\] cd /home/srvlan/.config/openbox
\[srvlan@srvlan:~$\] nano autostart.sh

et ajoutez les lignes ci-dessous en fin de fichier :

\# Lancement de l'agent d'authentification
/usr/lib/policykit-1-gnome/polkit-gnome-authentication-agent-1 &

# Affichage du fond d'écran
hsetroot -solid "#3399FF" &

# Lancement de conky
conky &

### 4 - Installation des utilitaires de VirtualBox

Ils permettront entre autres le copier/coller et l'accès au dossier partagé par le PC hôte.

Relevez au préalable la version du noyau linux :

\[srvlan@srvlan:~$\] uname -r

Retour = 4.19.0.x-686-pae

Installez en conséquence les paquets suivants :

\[srvlan@srvlan:~$\] sudo apt-get install dkms build-essential 
\[srvlan@srvlan:~$\] sudo apt-get install linux-headers-4.19.0-x-686-pae 

Accédez à présent au menu VirtualBox de la VM :  
\> Périphériques > Insérer l'image CD des Additions inv...

Montez le CD virtuel, installez les utilitaires et rebootez :

\[srvlan@srvlan:~$\] sudo mount /dev/cdrom /media/cdrom
\[srvlan@srvlan:~$\] cd /media/cdrom/
\[srvlan@srvlan:~$\] sudo ./VBoxLinuxAdditions.run
\[srvlan@srvlan:~$\] sudo reboot

Lancez ensuite le terminal et ajustez la taille de l'écran :

\[srvlan@srvlan:~$\] xrandr -s 1360x768

Vous pouvez aussi ajuster la taille à l'aide de la souris.

Sans fermer la VM, retirez l'ISO du lecteur CD virtuel.  
Menu Configuration de VirtualBox :  
\- - - Onglet Stockage  
\> Unités de stockage > VBoxGuestAdditions.iso  
\> Attributs > Cliquez sur l'icône CD  
\> Retirer le disque du lecteur virtuel > OK

Editez maintenant le fichier autostart.sh :

\[srvlan@srvlan:~$\] cd /home/srvlan/.config/openbox
\[srvlan@srvlan:~$\] nano autostart.sh

et ajoutez les lignes ci-dessous en fin de fichier :

\# Taille de l'écran au boot
xrandr -s 1360x768 &

### 5 - Modification des menus Openbox et Conky

_Nota : Le copier/coller peut à présent être utilisé._

#### _5.1 - Menu Openbox_

Editez le menu d'Openbox :

\[srvlan@srvlan:~$\] cd /etc/xdg/openbox
\[srvlan@srvlan:~$\] sudo mousepad menu.xml

et remplacez le contenu actuel du fichier par celui-ci :

<?xml version="1.0" encoding="UTF-8"?>
<openbox\_menu xmlns="http://openbox.org/"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://openbox.org/
file:///usr/share/openbox/menu.xsd">

<menu id="root-menu" label="Openbox 3">
<item label="Terminal \[xfce4-terminal\]">
<action name="Execute">
<execute>x-terminal-emulator</execute>
</action> 
</item>
<item label="Gestionnaire de fichiers \[pcmanfm\]">
<action name="Execute">
<execute>pcmanfm</execute>
</action>
</item>
<item label="Editeur de textes \[mousepad\]">
<action name="Execute">
<execute>mousepad</execute>
</action>
</item>
<item label="Editeur d'images \[gpicview\]">
<action name="Execute">
<execute>gpicview</execute>
</action>
</item>
<item label="Navigateur Web \[chromium\]">
<action name="Execute">
<execute>x-www-browser</execute>
</action>
</item>
<item label="Gestionnaire de paquets \[synaptic\]">
<action name="Execute">
<execute>synaptic-pkexec</execute>
</action>
</item>
<!-- Cela nécessite la présence du paquet 'menu' -->
<menu id="/Debian" />
<separator />
<menu id="client-list-menu" />
<separator />
<item label="Configurer Openbox">
<action name="Execute">
<execute>obconf-qt</execute>
</action>
</item>
<item label="Recharger la configuration Openbox ">
<action name="Reconfigure" />
</item>
<item label="Recharger la configuration du menu">
<action name="Restart" />
</item>
<separator />
<item label="Quitter">
<action name="Execute">
<execute>obsession-logout</execute>
</action>
</item>
</menu>
</openbox\_menu>

Enregistrez les modifications et quittez mousepad.

Copiez les 2 fichiers .xml dans le dossier de srvlan :

\[srvlan@srvlan:~$\] sudo cp /etc/xdg/openbox/\*.xml /home/srvlan/.config/openbox/ 

et modifiez les droits sur ceux-ci comme suit :

\[srvlan@srvlan:~$\] cd /home/srvlan/.config/openbox
\[srvlan@srvlan:~$\] sudo chown srvlan:srvlan \*  

Redémarrez le système :

\[srvlan@srvlan:~$\] sudo reboot

Une fois redémarré, testez les applications présentes dans le nouveau menu contextuel.

#### _5.2 - Menu Conky_

Traduisez conky en modifiant les lignes 65 à 78 de /etc/conky/conky.conf comme suit :

${color yellow}Démarré depuis:$color $uptime
${color yellow}Fréquence (en MHz):$color $freq
${color yellow}Température CPU:$color ${alignr}${acpitemp}°C
${color yellow}Charge RAM:$color $mem/$memmax - $mem...
${color yellow}Charge Swap:$color $swap/$swapmax - $swa...
${color yellow}Charge CPU:$color $cpu% ${cpubar 4}
${color yellow}Processus:$... $pro... ${color yellow}En cours:...
$hr
${color orange}Système de fichiers:
    / $color${fs\_used /}/${fs\_size /} ${fs\_bar 6 /}
${color orange}Débits réseau:
Sortant:$... ${... enp0s3} ${... orange} - Entrant:$... ${... enp0s3}
$hr
${color 00FF00} Nom PID CPU% MEM%

C'est fini, Openbox est en place, RAM utilisée < 150 Mo :

[![Capture - Bureau Openbox de Debian](/wp-content/uploads/2021/05/vm-srvlan-bureau-openbox-430x271.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/05/vm-srvlan-bureau-openbox.jpg)

Bureau Openbox de Debian

### 6 - Montage du dossier partagé par le PC hôte

Créez le dossier qui permettra d'afficher le contenu partagé par le PC hôte :

\[srvlan@srvlan:~$\] mkdir /home/srvlan/Partages

Le système d'initialisation de Debian 10 reposant sur systemd au lieu de sysVinit comme pour le réseau 2017, la procédure de montage automatique du dossier partagé diffère.

Créez le fichier de service home-srvlan-Partages.mount :

\[srvlan@srvlan:~$\] cd /etc/systemd/system  
\[srvlan@srvlan:~$\] sudo touch home-srvlan-Partages.mount

Editez celui-ci :

\[srvlan@srvlan:~$\] sudo nano home-srvlan-Partages.mount

et entrez le contenu suivant :

\[Unit\]
Description = Montage dossier partagé fourni par VirtualBox

\[Mount\]
What = _Entrez le nom du dossier partagé par le PC hôte_
Where = /home/srvlan/Partages
Type = vboxsf
Options=rw,uid=srvlan,gid=srvlan

\[Install\]
WantedBy = multi-user.target

Exemple pour What : What=Partage-Yohann

Intégrez le service dans la configuration de systemd :

\[srvlan@srvlan:~$\] sudo systemctl daemon-reload

et démarrez celui-ci :

\[srvlan@srvlan:~$\] sudo systemctl start home-srvlan-Partages.mount

Vérifiez son statut :

\[srvlan@srvlan:~$\] sudo systemctl status home-srvlan-Partages.mount

Si nécessaire, touche q pour quitter le résultat affiché.

Autorisez le lancement du service au boot de la VM :

\[srvlan@srvlan:~$\] sudo systemctl enable home-srvlan-Partages.mount

Un lien symbolique est créé.

Ouvrez pcmanfm et observer le contenu partagé par le PC hôte dans /home/srvlan/Partages.  

### 7 - Configuration du réseau

Attention, sous Debian 10, le nom des cartes réseau respecte une nouvelle nomenclature.

Vérifiez le à l'aide de la Cde suivante :

\[srvlan@srvlan:~$\] ip address

Résultat, eth0/eth1 sont renommées enp0s3/enp0s8 :

![Capture - Résultat de la Cde ip address](/wp-content/uploads/2019/04/srvlan-cde-ip-address.png)

Résultat de la Cde ip address

La VM qui sera placée à l'intérieur du réseau local virtuel impose de modifier le mode d'accès réseau de la carte enp0s3.

Pour cela, sans stopper la VM, accédez au menu Configuration de VirtualBox :  
\- - - Onglet Réseau  
\> Carte 1 > Mode d'accès réseau > Réseau interne  

#### _7.1 - Adressage IP fixe_

Editez le fichier de configuration réseau interfaces :

\[srvlan@srvlan:~$\] sudo nano /etc/network/interfaces

Commentez les lignes suivantes :

allow-hotplug enp0s3           _(ajoutez un # devant la ligne)_
iface enp0s3 inet dhcp        _(ajoutez un # devant la ligne)_

et ajoutez celles-ci :

auto enp0s3  
iface enp0s3 inet static  
address 192.168.2.2  
netmask 255.255.255.0  
network 192.168.2.0  
broadcast 192.168.2.255  
gateway 192.168.2.1  
  
auto enp0s8  
iface enp0s8 inet static  
address 192.168.3.1  
netmask 255.255.255.0  
network 192.168.3.0  
broadcast 192.168.3.255

Vous venez de créer 2 adresses IP fixes.

Relancez le service réseau :

\[srvlan@srvlan:~$\] sudo systemctl restart networking

et vérifiez le résultat avec la Cde ip address.  

#### _7.2 - Routage interne au serveur_

L'activation du routage avec la Cde ip\_forward permet, selon les règles définies dans la table de routage du serveur, le renvoi de paquets de données arrivés par une interface réseau vers une autre interface réseau.

Pour l'activez, éditez le fichier sysctl.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/sysctl.conf

et retirez le # de la ligne #net.ipv4.ip\_forward=1.

Relancez le service réseau :

\[srvlan@srvlan:~$\] sudo systemctl restart networking

#### _7.3 - Translation d'adresses NAT_

\-- Définition de WIKIBOOKS -- 
Objectif du NAT _(Network Address Translation)_ :  
Faire que les PC d'un réseau interne n'apparaissent que sous l'identifiant d'une seule IP pour les réseaux externes _(c'est un masquage ou IP Masquerading)_.

Votre box Internet _(passerelle)_ fait du NAT entre votre réseau privé et le réseau Internet et de ce fait votre fournisseur d'accès _(FAI)_ ne vous donne qu'une seule IP alors que vous pouvez très bien avoir plusieurs PC connectés à Internet à partir de votre réseau local.

Le NAT répond principalement au manque d'adresses IP dans le plan d'adressage ipV4 pour l'accès à Internet.

Il permet également de s'affranchir de la gestion des tables de routage et fonctionne avec le service iptables installé par défaut avec Debian.

Vérifiez l'état du NAT ou IP Masquerading :

\[srvlan@srvlan:~$\] sudo iptables -L -t nat

Résultat, le NAT est vu comme inactif :

![Capture - Le NAT (translation d'adresses) est vu non activé](/wp-content/uploads/2018/03/iptables_nat_inactif.png)

Le NAT (translation d'adresses) est vu non activé

Activez celui-ci :

\[srvlan@srvlan:~$\] sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

et affichez de nouveau la table NAT :

\[srvlan@srvlan:~$\] sudo iptables -L -t nat

Résultat, le NAT est actif _(ligne en jaune)_ :

![Capture - Le NAT est vu activé](/wp-content/uploads/2018/03/iptables_nat_actif.png)

Le NAT (translation d'adresses) est vu activé

Effectuez une sauvegarde de la règle NAT activée :

\[srvlan@srvlan:~$\] sudo bash -c "iptables-save > /etc/iptables\_regles.save"

Editez le fichier de configuration réseau interfaces :

\[srvlan@srvlan:~$\] sudo nano /etc/network/interfaces

et ajoutez ces 2 lignes en fin de fichier :

\# Iptables - Règle de translation d'adresses NAT
post-up iptables-restore < /etc/iptables\_regles.save

La règle sera appliquée à chaque démarrage de la VM.

Supprimez maintenant celle-ci et affichez la table NAT :

\[srvlan@srvlan:~$\] sudo iptables -F -t nat  
\[srvlan@srvlan:~$\] sudo iptables -L -t nat

Vous devez constater la disparition de la règle.

Relancez le service réseau :

\[srvlan@srvlan:~$\] sudo systemctl restart networking

Affichez de nouveau la table NAT :

\[srvlan@srvlan:~$\] sudo iptables -L -t nat

Vous devez constater la réapparition de la règle.

Lecture de la table de routage avec la Cde ip r :

![Capture - Table de routage de srvlan](/wp-content/uploads/2019/04/srvlan_table_routage.png)

Table de routage de srvlan

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
C'est terminé !  
Le premier serveur virtuel est créé.  
Le mémento 2.1 vous attend pour  
l'ajout d'un serveur IPFire.

[Mémento 2.1](/srvsec-ipfire-debian10)
