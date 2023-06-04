---
title: "srvdmz - VirtualBox / Debian 10"
date: "2022-04-28"
categories: 
  - "archives"
---

## Mémento 3.1 - Serveur srvdmz

Vous allez maintenant créer la VM srvdmz que vous placerez en zone dite démilitarisée et qui fournira plus tard des services Web, FTP, DNS, Mail, etc... pour Internet et le réseau local.

La configuration ressemblera à celle de la VM srvlan hormis la partie réseau.

### 1 - Construction de la VM depuis VirtualBox

L'utilisation de VirtualBox est considérée acquise.

A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](/virtualbox-installation/)  
[VirtualBox - Mode d’accès réseau par pont](/virtualbox-pont-reseau/)

#### _1.1 - Création et configuration_

Téléchargez l'ISO 32 bits debian-10.x.y-i386-netinst.iso :  
[https://cdimage.debian.org/.../10.10.0/i386/iso-cd/](https://cdimage.debian.org/cdimage/archive/10.10.0/i386/iso-cd/)

Démarrez l'application VirtualBox, puis :

Menu Nouvelle :  
\- Nom srvdmz DMZ - Type Linux - Version Debian (32-b...  
\- Taille de la mémoire > 512 Mo  
\- Disque dur > Créer un disque dur virtuel maintenant  
\- Type de fichier de disque dur > VDI  
\- Stockage sur disque dur ... > Dynamiquement alloué  
\- Emplacement du fichier et taille > 12 Go > Créer

La VM est créée dans le panneau gauche de VirtualBox.

Sélectionnez la nouvelle VM, puis :

Menu Configuration :  
\- - - Onglet Général  
\> Avancé > Presse-papier partagé  
\> Bidirectionnel  
  
\- - - Onglet Système  
\> Carte mère > Ordre d'amorçage > Décochez Disquette  
\> Carte mère > Fonctions avancées > Cochez IO-APIC  
\> Processeur > 2 CPU et cochez Activer PAE/NX  
  
\- - - Onglet Affichage  
\> Ecran > Contrôleur graphique > VMSVGA  
  
\- - - Onglet Stockage  
\> Unités de stockage > Sélectionnez Vide  
\> Attributs > Cliquez sur l'icône CD  
\> Sélectionnez Choisissez un fichier de disque ...  
\> Entrez le chemin de l'image ISO Debian > Ouvrir

Facultatif, accès au dossier partagé par le PC hôte.  
\- - - Onglet Dossiers partagés  
\> Cliquez sur l'icône + > Ajouter un dossier partagé  
\> Chemin du dossier > Sélectionnez Autre...  
\> Accédez à votre dossier > Ex : C:\\Partage-Windows  
\> Sélectionner un dossier > OK > OK

Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de la distribution Debian_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

Menu Démarrer :  
La VM s'exécute, contrôlez l'ISO affichée et démarrez.

Sélectionnez Graphical Install et appliquez ce qui suit :  
\- Language > French  
\- Pays (territoire ou région) > France  
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
\- Table des partitions > Terminer le partitionnement ...  
\- Faut-il appliquer les changements … disques ? > Oui

L'installation de base commence :  
\- Faut-il analyser un autre CD ou DVD > Non  
\- Pays du miroir de l'archive Debian > France  
\- Miroir de l'archive Debian > ftp.fr.debian.org  
\- Mandataire HTTP (lais...) > Laissez vide

L'installation continue :  
\- Souhaitez-vous participer à l'étude statistique... > Non  
\- Logiciels à installer  
\> Décochez "environnement de bureau Debian"  
\> Décochez "serveur d'impression"  
\> Conservez "utilitaires usuels du système"

L'installation se termine :  
\- Installer ... de démarrage GRUB sur le secteur ... > Oui  
\- Périphérique ... programme de démarrage > /dev/sda  
\- Installation terminée > Continuer _(sans retrait du CD)_

Le système reboot :  
\- srvdmz login : srvdmz  
\- Password : Votre MDP srvdmz  
Si tout est OK, affichage du prompt \[srvdmz@srvdmz:~$\]

Connectez-vous à présent comme administrateur root :

\[srvdmz@srvdmz:~$\] su root  
Mot de passe : Votre MDP root  
\[root@srvdmz:~#\]

#### _1.3 - Gestion des paquets Debian_

Dans le but de garder un Debian minimaliste, empêchez les paquets recommandés/suggérés par le gestionnaire de paquets apt d'être installés par défaut comme des dépendances.

Créez et éditez le fichier 00pas-de-recommends :

\[root@srvdmz:~#\] cd /etc/apt
\[root@srvdmz:~#\] touch apt.conf.d/00pas-de-recommends
\[root@srvdmz:~#\] nano apt.conf.d/00pas-de-recommends

Entrez ensuite ces 2 lignes et enregistrez le fichier :

APT::Install-Recommends "false";  
APT::Install-Suggests "false";

Touches CTRL + x pour enregistrer > O = Oui > Entrée

#### _1.4 - Installation de sudo et MAJ de Debian_

Autorisez l'utilisateur srvdmz à exécuter des Cdes en tant que root à l'aide du programme sudo. Vous diminuerez ainsi, connecté en tant que srvdmz, le risque de casser la VM par erreur.

Installez sudo et ajoutez srvdmz dans le groupe sudo :

\[root@srvdmz:~#\] apt-get install sudo  
\[root@srvdmz:~#\] sudo adduser srvdmz sudo 

Redémarrez le système pour une prise en compte :

\[root@srvdmz:~#\] sudo reboot

et connectez-vous en tant qu'utilisateur srvdmz :

[![Capture - Premier démarrage de Debian ](/wp-content/uploads/2019/04/srvdmz-premier-demarrage-430x370.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/04/srvdmz-premier-demarrage.png)

Premier démarrage de Debian

Mettez à jour la distribution Debian :

\[srvdmz@srvdmz:~$\] sudo apt-get update  
\[srvdmz@srvdmz:~$\] sudo apt-get upgrade  
\[srvdmz@srvdmz:~$\] sudo apt-get dist-upgrade

Pour info, la VM peut être arrêtée de 2 façons :

\[srvdmz@srvdmz:~$\] sudo poweroff  
\[srvdmz@srvdmz:~$\] sudo shutdown -h now 

Stoppez donc celle-ci, réduisez depuis VirtualBox la mémoire allouée à 384 Mo et redémarrez.

### 2 - Ajout du serveur graphique Xorg

Vous devez, comme préalable à la mise en place d'Openbox, installer un serveur graphique.

Installez xorg à l'aide des Cdes suivantes :

\[srvdmz@srvdmz:~$\] sudo apt-get install xfonts-base   
\[srvdmz@srvdmz:~$\] sudo apt-get install xserver-xorg    
\[srvdmz@srvdmz:~$\] sudo apt-get install x11-xserver-utils  

### 3 - Ajout du gestionnaire de fenêtres Openbox

#### _3.1 - Installation de base_

Entrez la Cde suivante :

\[srvdmz@srvdmz:~$\] sudo apt-get install openbox obmenu obconf-qt obconf-qt-l10n

#### _3.2 - Gestionnaire de session, terminal_ ...

Ajoutez ces 3 programmes avec les Cdes suivantes :

\[srvdmz@srvdmz:~$\] sudo apt-get install slim xfce4-terminal   
\[srvdmz@srvdmz:~$\] sudo apt-get install chromium chromium-l10n chromium-sandbox

Redémarrez le système :

\[srvdmz@srvdmz:~$\] sudo reboot

Le gestionnaire de session/connexion slim s'ouvre.

Entrez le nom d'utilisateur srvdmz > Touche Entrée.  
Entrez ensuite son MDP > Touche Entrée.

Le bureau s'affiche, effectuez un clic droit sur celui-ci pour activer le menu contextuel d'Openbox et vérifiez le fonctionnement de Terminal emulator et Web browser.

Info sur le thème actif du gestionnaire de session slim :  
\- Choix dans /etc/slim.conf, paramètre current\_theme.  
\- Thèmes disponibles dans /usr/share/slim/themes.

#### _3.3 - Barre des tâches_

Ouvrez Terminal emulator et installez la barre tint2 :

\[srvdmz@srvdmz:~$\] sudo apt-get install tint2

Créez un dossier openbox dans /home/srvdmz/.config :

\[srvdmz@srvdmz:~$\] mkdir /home/srvdmz/.config/openbox

Créez dans le dossier un fichier autostart.sh :

\[srvdmz@srvdmz:~$\] cd /home/srvdmz/.config/openbox
\[srvdmz@srvdmz:~$\] touch autostart.sh

Editez celui-ci :

\[srvdmz@srvdmz:~$\] nano autostart.sh

et insérez les lignes suivantes avant d'enregistrer :

\# Lancement de la barre des tâches  
tint2 &

Redémarrez et découvrez la barre des tâches tint2 :

\[srvdmz@srvdmz:~$\] sudo reboot

#### _3.4 - Assistant du gestionnaire de session_

Ouvrez le terminal et installez l'outil obsession :

\[srvdmz@srvdmz:~$\] sudo apt-get install obsession

Editez le menu contextuel utilisé par Openbox :

\[srvdmz@srvdmz:~$\] sudo nano /etc/xdg/openbox/menu.xml

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

\- Ouvrez ensuite ce menu _(clic droit sur le bureau)_  
\> Exit pour observer le menu fourni de base  
\> Annuler

\- Ouvrez de nouveau le menu contextuel  
\> Restart pour traiter la modification ci-dessus

\- Ouvrez une dernière fois le menu contextuel  
\> Exit pour voir le nouveau menu fourni par obsession  
\> Annuler

Des boutons Arrêter, Redémarrer, etc... compléteront le menu une fois le paquet policykit-1-gnome installé.

#### _3.5 - Utilitaires pratiques_

Installez les 7 utilitaires suivants :  
\- Le gestionnaire de paquets synaptic  
\- L'éditeur de textes léger mousepad  
\- Le gestionnaire de fichiers pcmanfm  
\- Le créateur de fond d'écran hsetroot  
\- L'agent d'authentification policykit-1-gnome  
\- Le moniteur système conky  
\- L'éditeur d'images gpicview

\[srvdmz@srvdmz:~$\] sudo apt-get install synaptic mousepad
\[srvdmz@srvdmz:~$\] sudo apt-get install pcmanfm hsetroot
\[srvdmz@srvdmz:~$\] sudo apt-get install policykit-1-gnome 
\[srvdmz@srvdmz:~$\] sudo apt-get install conky gpicview  

Prévoyez d'en exécuter 3 au démarrage du système.

Editez pour cela le fichier autostart.sh :

\[srvdmz@srvdmz:~$\] cd /home/srvdmz/.config/openbox
\[srvdmz@srvdmz:~$\] nano autostart.sh

et ajoutez les lignes ci-dessous en fin de fichier :

\# Lancement de l'agent d'authentification
/usr/lib/policykit-1-gnome/polkit-gnome-authentication-agent-1 &

# Affichage du fond d'écran
hsetroot -solid "#996600" &

# Lancement de conky
conky &

### 4 - Installation des utilitaires de VirtualBox

Ils permettront entre autres le copier/coller et l'accès au dossier partagé par le PC hôte.

Relevez au préalable la version du noyau linux :

\[srvdmz@srvdmz:~$\] uname -r

Retour = 4.19.0.x-686-pae

Installez en conséquence les paquets suivants :

\[srvdmz@srvdmz:~$\] sudo apt-get install dkms build-essential 
\[srvdmz@srvdmz:~$\] sudo apt-get install linux-headers-4.19.0-x-686-pae 

Accédez à présent au menu VirtualBox de la VM :  
\> Périphériques > Insérer l'image CD des Additions inv...

Montez le CD virtuel, installez les utilitaires et rebootez :

\[srvdmz@srvdmz:~$\] sudo mount /dev/cdrom /media/cdrom/
\[srvdmz@srvdmz:~$\] cd /media/cdrom/
\[srvdmz@srvdmz:~$\] sudo ./VBoxLinuxAdditions.run
\[srvdmz@srvdmz:~$\] sudo reboot

Lancez ensuite le terminal et ajustez la taille de l'écran :

\[srvdmz@srvdmz:~$\] xrandr -s 1360x768

Vous pouvez aussi ajuster la taille à l'aide de la souris.

Sans fermer la VM, retirez l'ISO du lecteur CD virtuel.  
Menu Configuration de Virtualbox :  
\- - - Onglet Stockage  
\> Unités de stockage > VBoxGuestAdditions.iso  
\> Attributs > Cliquez sur l'icône CD  
\> Retirer le disque du lecteur virtuel > OK

Editez maintenant le fichier autostart.sh :

\[srvdmz@srvdmz:~$\] cd /home/srvdmz/.config/openbox
\[srvdmz@srvdmz:~$\] nano autostart.sh

et ajoutez les lignes ci-dessous en fin de fichier :

\# Taille de l'écran au boot
xrandr -s 1360x768 &

### 5 - Modification des menus Openbox et Conky

_Nota : Le copier/coller peut à présent être utilisé._

#### _5.1 - Menu Openbox_

Editez le menu d'Openbox :

\[srvdmz@srvdmz:~$\] sudo mousepad /etc/xdg/openbox/menu.xml

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
<separator /
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

Copiez les 2 fichiers .xml dans le dossier srvdmz :

\[srvdmz@srvdmz:~$\] sudo cp /etc/xdg/openbox/\*.xml /home/srvdmz/.config/openbox/ 

et modifiez les droits sur ceux-ci comme suit :

\[srvdmz@srvdmz:~$\] cd /home/srvdmz/.config/openbox
\[srvdmz@srvdmz:~$\] sudo chown srvdmz:srvdmz \* 

Redémarrez le système :

\[srvdmz@srvdmz:~$\] sudo reboot

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
${color 00FF00}Nom PID CPU% MEM%

C'est fini, Openbox est en place, RAM utilisée < 150 Mo :

[![Capture - Bureau Openbox de Debian](/wp-content/uploads/2021/05/vm-srvdmz-bureau-openbox-430x271.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/05/vm-srvdmz-bureau-openbox.jpg)

Bureau Openbox de Debian

### 6 - Montage du dossier partagé par le PC hôte

Créez le dossier qui permettra d'afficher le contenu partagé par le PC hôte :

\[srvdmz@srvdmz:~$\] mkdir /home/srvdmz/Partages

Le système d'initialisation de Debian 10 reposant sur systemd au lieu de sysVinit comme pour le réseau 2014-2017, la procédure de montage automatique du dossier partagé diffère.

Créez le service home-srvdmz-Partages.mount :

\[srvdmz@srvdmz:~$\] cd /etc/systemd/system  
\[srvdmz@srvdmz:~$\] sudo touch home-srvdmz-Partages.mount

Editez celui-ci :

\[srvdmz@srvdmz:~$\] sudo nano home-srvdmz-Partages.mount

et entrez le contenu suivant :

\[Unit\]
Description = Montage dossier partagé fourni par VirtualBox

\[Mount\]
What = _Entrez le nom du dossier partagé par le PC hôte_
Where = /home/srvdmz/Partages
Type = vboxsf
Options=rw,uid=srvdmz,gid=srvdmz

\[Install\]
WantedBy = multi-user.target

Exemple pour What : What=Partage-Yohann

Intégrez le service dans la configuration de systemd :

\[srvdmz@srvdmz:~$\] sudo systemctl daemon-reload

et démarrez celui-ci :

\[srvdmz@srvdmz:~$\] sudo systemctl start home-srvdmz-Partages.mount

Vérifiez son statut :

\[srvdmz@srvdmz:~$\] sudo systemctl status home-srvdmz-Partages.mount

Si nécessaire, touche q pour quitter le résultat affiché.

Autorisez le lancement du service au boot de la VM :

\[srvdmz@srvdmz:~$\] sudo systemctl enable home-srvdmz-Partages.mount

Un lien symbolique est créé.

Ouvrez pcmanfm et observez le contenu partagé par le PC hôte dans /home/srvdmz/Partages.

### 7 - Configuration du réseau

Attention, sous Debian 10, le nom des cartes réseau respecte une nouvelle nomenclature.

Vérifiez le à l'aide de la Cde suivante :

\[srvdmz@srvdmz:~$\] ip address

Résultat, eth0 est renommée enp0s3 :

![Capture - Résultat Cde ip address](/wp-content/uploads/2019/04/srvdmz-cde-ip-address.png)

Résultat de la Cde ip address

La VM qui sera placée à l'intérieur du réseau local virtuel impose de modifier le mode d'accès réseau de la carte enp0s3.

Pour cela, sans stopper la VM, accédez au menu Configuration de VirtualBox :  
\- - - Onglet Réseau  
\> Carte 1 > Mode d'accès réseau > Réseau interne

#### _7.1 - Adressage IP fixe_

Editez le fichier de configuration réseau interfaces :

\[srvdmz@srvdmz:~$\] sudo nano /etc/network/interfaces

Commentez les lignes suivantes :

allow-hotplug enp0s3 _(ajoutez un # devant la ligne)_ 
iface enp0s3 inet dhcp _(ajoutez un # devant la ligne)_ 

et ajoutez celles-ci :

auto enp0s3  
iface enp0s3 inet static  
address 192.168.4.2  
netmask 255.255.255.0  
network 192.168.4.0  
broadcast 192.168.4.255  
gateway 192.168.4.1

Vous venez d'appliquer une adresse IP fixe à enp0s3.

Relancez le service réseau :

\[srvdmz@srvdmz:~$\] sudo systemctl restart networking

et vérifiez le résultat avec la Cde ip address.

Affichez ensuite la table de routage avec la Cde ip r :

![Capture - Table de routage de srvdmz](/wp-content/uploads/2019/04/srvdmz_table_routage.png)

Table de routage de srvdmz

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Bien !  
Les serveurs sont prêts. Le mémento  
4.1 vous attend pour la création des  
VM clientes du réseau local virtuel.

[Mémento 4.1](/virtualbox-clients-debian10-creation/)
