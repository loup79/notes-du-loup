---
title: "Open vSwitch / Debian 10"
date: "2022-04-26"
categories: 
  - "archives"
---

## Mémento 5.1 - Switch virtuel

Open vSwitch _(OVS)_ se comportera comme un switch Ethernet en proposant la gestion des adresses MAC tel un switch L2 ainsi que celle des adresses IP pour le routage des paquets IP tel un switch L3. 
  
Le switch virtuel sera, contrairement à une architecture standard, installé sur une VM Debian et non sur le PC hôte de l'hyperviseur VirtualBox.  
  
La gestion possible de VLAN ne sera pas traitée ici.

### 1 - Construction de la VM depuis VirtualBox

L'utilisation de VirtualBox est considérée acquise.  
  
A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](/virtualbox-installation/)  
[VirtualBox - Mode d’accès réseau par pont](/virtualbox-pont-reseau/)

#### _1.1 - Création et configuration_

Téléchargez l'ISO debian-10.x.y-amd64\-netinst.iso :  
[https://cdimage.debian.org/.../current/amd64/iso-cd/](https://cdimage.debian.org/cdimage/archive/10.10.0/amd64/iso-cd/)

Démarrez ensuite l'application VirtualBox, puis :

Menu Nouvelle de VirtualBox :  
\- Nom ovs - Type Linux - Version Debian (64-bit)  
\- Taille de la mémoire > 1024 Mo _(Temporairement)_  
\- Disque dur > Créer un disque dur virtuel maintenant  
\- Type de fichier de disque dur > VDI  
\- Stockage sur disque dur ... > Dynamiquement alloué  
\- Emplacement du fichier et taille > 8 Go > Créer

La VM est créée dans le panneau gauche de VirtualBox.

Sélectionnez la nouvelle VM, puis :

Menu Configuration de VirtualBox :  
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
  
\> OK

Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de la distribution Debian_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparait lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

Menu Démarrer de VirtualBox :  
La VM s'exécute, contrôlez l'ISO affichée et démarrez.

Sélectionnez Graphical Install et appliquez ce qui suit :  
\- Language > Français  
\- Pays (territoire ou région) > France  
\- Disposition de clavier à utiliser > Français  
\- Nom de machine > ovs  
\- Domaine > Laissez le champ vide  
\- MDP du super utilisateur root > Votre MDP root  
\- Confirmation du MDP > Votre MDP root  
\- Nom complet du nouvel utilisateur > Ex: switch  
\- Identifiant pour le compte utilisateur > switch  
\- MDP pour le nouvel utilisateur > Votre MDP switch  
\- Confirmation du MDP > Votre MDP switch  
\- Méthode de partitionnement > Assisté - utili... entier  
\- Disque à partitionner > Celui proposé de 8 Go  
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
\> Décochez environnement de bureau Debian  
\> Décochez serveur d'impression  
\> Conservez utilitaires usuels du système

L'installation se termine :  
\- Installer ... de démarrage GRUB sur le secteur ... > Oui  
\- Périphérique ... programme de démarrage > /dev/sda  
\- Installation terminée > Continuer _(sans retrait du CD)_

Le système reboot :  
\- ovs login : switch  
\- Password : Votre MDP switch  
Si tout est OK, affichage du prompt \[switch@ovs:~$\]

Connectez-vous à présent comme administrateur root :

\[switch@ovs:~$\] su root
Mot de passe : Votre MDP root
\[root@ovs:~#\]

#### _1.3 - Installation de sudo et MAJ de Debian_

Autorisez l'utilisateur switch à exécuter des Cdes en tant que root à l'aide du programme sudo. Vous diminuerez ainsi, connecté en tant que switch, le risque de casser la VM par erreur.

Installez sudo et ajoutez switch dans le groupe sudo :

\[root@ovs:~#\] apt-get install sudo  
\[root@ovs:~#\] sudo adduser switch sudo 

Redémarrez le système pour une prise en compte :

\[root@ovs:~#\] sudo reboot

et connectez-vous en tant qu'utilisateur switch :

![Capture - Open vSwitch : Premier démarrage de la VM ovs](/wp-content/uploads/2019/05/ovs-premier-demarrage-1.png)

Connexion de l'utilisateur switch sur la VM ovs

Mettez à jour la distribution Debian :

\[switch@ovs:~$\] sudo apt-get update  
\[switch@ovs:~$\] sudo apt-get upgrade  
\[switch@ovs:~$\] sudo apt-get dist-upgrade

Pour info, la VM peut être arrêtée de 2 façons :

\[switch@ovs:~$\] sudo poweroff  
\[switch@ovs:~$\] sudo shutdown -h now 

Stoppez donc celle-ci, réduisez depuis VirtualBox la mémoire allouée à 384 Mo et redémarrez.

### 2 - Installation et configuration d'Open vSwitch

Au préalable, observez la configuration réseau active :

![Capture - Open vSwitch : Interfaces réseau de base](/wp-content/uploads/2019/04/ovs-interfaces-reseau-1.png)

Interfaces réseau de base soit lo et enp0s3

#### _2.1 - Installation_

Installez les paquets nécessaires à Open vSwitch :

\[switch@ovs:~$\] sudo  apt-get install net-tools 
\[switch@ovs:~$\] sudo apt-get install openvswitch-switch 
\[switch@ovs:~$\] sudo reboot 

Des Cdes incluses dans net-tools sont utilisées par des scripts openvswitch.

Contrôlez la version installée d'Open vSwitch :

\[switch@ovs:~$\] sudo ovs-vsctl show 

![Capture - Open vSwitch : Affichage de la version](/wp-content/uploads/2019/04/ovs-version-openvswitch.png)

Version Open vSwitch

et l'intégration de son module dans le noyau linux :

\[switch@ovs:~$\] sudo modinfo openvswitch 

[![Capture - Open vSwitch : Informations module openvswitch](/wp-content/uploads/2019/04/ovs-module-openvswitch-430x280.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/04/ovs-module-openvswitch.png)

Informations sur le module d'Open vSwitch

\- - Liste des principaux composants OVS installés - -

/usr/lib/openvswitch-common/ovs-vswitchd :  
Service de commutation _(compatible OpenFlow)_.

/usr/bin/ovs-appctl :  
Configuration CLI _(Command Line Interface)_ du service.

/usr/bin/ovs-vsctl :  
Configuration CLI du service via le serveur de Bdd OVS.

/usr/sbin/ovsdb-server :  
Bdd contenant la configuration au niveau commutateur.

/usr/bin/ovsdb-client :  
Outil de dialogue CLI avec le serveur de Bdd OVS.

/usr/bin/ovsdb-tool :  
Configuration CLI des fichiers de la Bdd.

/usr/bin/ovs-dpctl :  
Configuration CLI du module noyau d'Open vSwitch.

/usr/bin/ovs-ofctl :  
Utilitaire CLI de contrôle des commutateurs OpenFlow.

La Bdd conf.db se situe dans /etc/openvswitch/.  
Les logs se situent dans /var/log/openvswitch/.

Contrôlez l'activation du service openvswitch-switch :

\[switch@ovs:~$\] sudo systemctl status openvswitch-switch

![Capture - Open vSwitch : Vue activation du service openvswitch-switch](/wp-content/uploads/2019/04/ovs-service-openvswitch-switch.png)

Service openvswitch-switch activé

#### _2.2 - Configuration_

Commencez par créez un bridge _(switch)_ de nom br0 :

\[switch@ovs:~$\] sudo ovs-vsctl add-br br0   
\[switch@ovs:~$\] sudo ovs-vsctl show    

![Capture - Open vSwitch : Vue création bridge br0](/wp-content/uploads/2019/04/ovs-creation-br0.png)

Création du bridge br0

Un port et une interface virtuels de même nom ont été associés au bridge.

Utilisez la Cde del-br pour détruire un bridge existant.

Observez les infos détaillées du bridge br0 comme suit :

\[switch@ovs:~$\] sudo ovs-vsctl list br br0      

et la création de l'interface br0 avec la Cde ip address :

![Capture - Open vSwitch : Vue création interface réseau br0](/wp-content/uploads/2019/04/ovs-interfaces-reseau-2.png)

Cde ip address montrant l'interface br0 créée

Rattachez à présent l'interface enp0s3 au bridge br0 :

\[switch@ovs:~$\] sudo ovs-vsctl add-port br0 enp0s3  
\[switch@ovs:~$\] sudo ovs-vsctl show

![Capture - Open vSwitch : Vue affectation port et interface enp0s3 sur br0](/wp-content/uploads/2019/04/ovs-affectation-port-enps03.png)

Ajout du port enp0s3 sur br0

Le port virtuel enp0s3 et l'interface de réseau virtuel enp0s3 ne font qu'un.

Utilisez la Cde del-port pour détruire un port existant.

Observez de nouveau le résultat de la Cde ip address :

![Capture - Open vSwitch : Vue adresses MAC enp0s3 et br0 identiques](/wp-content/uploads/2019/04/ovs-affectation-interface-enp0s3.png)

Adresses MAC de enp0s3 et br0 identiques

L'interface br0 possède à présent la même adresse MAC que enp0s3.

### 3 - Intégration de la VM ovs dans le réseau virtuel

#### _3.1 - Configuration réseau depuis VirtualBox_

Stoppez la VM.

\[switch@ovs:~$\] sudo poweroff

Accédez au menu Configuration de VirtualBox, puis :  
\- - - Onglet Réseau  
\> Interface 1  
\> Mode d'accès réseau > Réseau interne  
\> Nom > Sélectionnez switch\_interne  
\> Avancé > Mode Promiscuité  
\> Sélectionnez Autoriser les VMs  
  
\> Interface 2  
\> Cochez Activer l'interface réseau  
\> Mode d'accès réseau > Réseau interne  
\> Nom > Entrez liaison\_vm1  
\> Avancé > Mode Promiscuité  
\> Sélectionnez Autoriser les VMs  
  
\> Interface 3  
\> Cochez Activer l'interface réseau  
\> Mode d'accès réseau > Réseau interne  
\> Nom > Entrez liaison\_vm2  
\> Avancé > Mode Promiscuité  
\> Sélectionnez Autoriser les VMs  
  
\> OK

Redémarrez la VM.

Il est impossible, OVS n'étant pas installé sur le PC hôte de VirtualBox, d'attribuer aux interfaces 2 et 3 un mode d'accès réseau généralement utilisé avec OVS soit un mode de type Accès par pont au travers d'une interface réseau virtuelle TAP _([Voir le § 4](#4_-_Raccordement_des_2_clients_Debian_sur_OVS))_.  
  
C'est donc le type Réseau interne qui sera utilisé.  
  
Le mode promiscuité empêche une interface réseau de rejeter les trames autres que celles qui lui sont destinées _(mode par défaut)_.  
  
Il est nécessaire, dans une infrastructure virtualisée telle Open vSwitch installé sur une VM, d'appliquer le mode promiscuité à une interface réseau devant agir comme un pont _(bridge)_.

#### _3.2 - Modification du fichier réseau de Debian_

Vous allez à présent configurer OVS au boot de la VM.

Editez maintenant le fichier réseau interfaces :

\[switch@ovs:~$\] sudo nano /etc/network/interfaces

et modifiez le comme suit :

\# This file describes the network interfaces available on ...
# and how to activate them. For more information, see ...

source /etc/network/interfaces.d/\*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
# Mettre un # devant les 2 lignes ci-dessous
#allow-hotplug enp0s3
#iface enp0s3 inet dhcp

# Configuration Open vSwitch
allow-ovs br0
iface br0 inet static
address 192.168.3.15
netmask 255.255.255.0
gateway 192.168.3.1
ovs\_type OVSBridge
ovs\_ports enp0s3

allow-br0 enp0s3
iface enp0s3 inet manual
ovs\_bridge br0
ovs\_type OVSPort

Redémarrez ensuite le service réseau :

\[switch@ovs:~$\] sudo systemctl restart networking

et contrôlez le résultat avec la Cde ip address :

![Capture - Open vSwitch : Vue adresse IP sur interface réseau br0](/wp-content/uploads/2019/04/ovs-adresse-ip-br0.png)

Contrôle des interfaces réseau

Constat :  
\- 2 nouvelles interfaces réseau enp0s8 et enp0s9  
\- L'interface br0 a la même adresse MAC que enp0s3  
\- L'interface br0 possède l'adresse IP 192.168.3.15  
\- Le ping vers l'adresse lP 192.168.3.1 fonctionne

L'adresse IP 192.168.3.15 sera utilisée plus tard pour administrer Open vSwitch à distance.

### 4 - Raccordement des 2 clients Debian sur OVS

Open vSwitch est généralement installé sur le PC hôte de l'hyperviseur. Un bridge tel br0 utilise alors les interfaces réseau physiques du PC hôte pour l'accès à Internet et des interfaces réseau virtuelles TAP de niveau L2 pour l'accès aux VM.

Dans le cas présent, Open vSwitch est installé sur une VM de VirtualBox.

Vous allez donc raccorder les 2 clients Debian sur les interfaces réseau enp0s8/9 de la VM ovs, interfaces qui seront rattachées à des bridges de nom br1/2.

Vous lierez les 3 bridges à l'aide de ports patch :

![Image - Open vSwitch : Bridges en cascade](/wp-content/uploads/2019/04/ovs-patch-bridges.png)

Schéma : Bridges br0, br1 et br2 montés en cascade

L'ensemble ainsi raccordé _(ports patch)_ peut être vu comme un seul pont.  
La création de VLAN et le routage de paquets IP entre ceux-ci restent possibles.

#### _4.1 - Création des bridges et des ports patch_

Créez les bridges br1 et br2 :

\[switch@ovs:~$\] sudo ovs-vsctl add-br br1  
\[switch@ovs:~$\] sudo ovs-vsctl add-br br2 

Rattachez les interfaces enp0s8/9 aux bridges br1/2 :

\[switch@ovs:~$\] sudo ovs-vsctl add-port br1 enp0s8  
\[switch@ovs:~$\] sudo ovs-vsctl add-port br2 enp0s9  

Reliez maintenant br0 avec br1 :

\[switch@ovs:~$\] sudo ovs-vsctl -- add-port br0 br0-patch0 \\
-- set interface br0-patch0 type=patch options:peer=br1-patch0
 
\[switch@ovs:~$\] sudo ovs-vsctl -- add-port br1 br1-patch0 \\
-- set interface br1-patch0 type=patch options:peer=br0-patch0   

Le caractère \\ indique d'écrire le tout sur une seule ligne.  

puis br1 avec br2 :

\[switch@ovs:~$\] sudo ovs-vsctl -- add-port br1 br1-patch1 \\
-- set interface br1-patch1 type=patch options:peer=br2-patch0 
 
\[switch@ovs:~$\] sudo ovs-vsctl -- add-port br2 br2-patch0 \\
-- set interface br2-patch0 type=patch options:peer=br1-patch1  

Contrôlez la prise en compte de la configuration :

\[switch@ovs:~$\] sudo ovs-vsctl  show

![Capture - Open vSwitch : Contrôle configuration](/wp-content/uploads/2021/10/ovs-controle-patch-bridges.png)

Open vSwitch : Contrôle de la configuration

#### _4.2 - Configuration réseau depuis VirtualBox_

Sans stopper les VM, modifiez l'onglet réseau des 2 clients debian10-vm\* comme suit :  
\> Interface 1  
\> Mode d'accès réseau > Réseau interne  
\> Nom > Sélectionnez liaison\_vm\* selon la VM

#### _4.3 - Modification du fichier réseau de Debian_

Modifiez le fichier interfaces de la VM ovs comme suit :

\# This file describes the network interfaces available on ...
# and how to activate them. For more information, see ...

source /etc/network/interfaces.d/\*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
# Mettre un # devant les 2 lignes ci-dessous
#allow-hotplug enp0s3
#iface enp0s3 inet dhcp

# Configuration Open vSwitch
allow-ovs br0
iface br0 inet static
address 192.168.3.15
netmask 255.255.255.0
gateway 192.168.3.1
ovs\_type OVSBridge
ovs\_ports enp0s3 br0-patch0

allow-br0 enp0s3
iface enp0s3 inet manual
ovs\_bridge br0
ovs\_type OVSPort

allow-br0 br0-patch0
iface br0-patch0 inet manual
ovs\_bridge br0
ovs\_type OVSPatchPort
ovs\_patch\_peer br1-patch0  
  
allow-ovs br1
iface br1 inet manual
ovs\_type OVSBridge
ovs\_ports enp0s8 br1-patch0 br1-patch1
  
allow-br1 enp0s8
iface enp0s8 inet manual
ovs\_bridge br1
ovs\_type OVSPortallow br1
 
allow-br1 br1-patch0
iface br1-patch0 inet manual
ovs\_bridge br1
ovs\_type OVSPatchPort
ovs\_patch\_peer br0-patch0
 
allow-br1 br1-patch1
iface br1-patch1 inet manual
ovs\_bridge br1
ovs\_type OVSPatchPort
ovs\_patch\_peer br2-patch0
  
allow-ovs br2
iface br2 inet manual
ovs\_type OVSBridge
ovs\_ports enp0s9 br2-patch0
  
allow-br2 enp0s9
iface enp0s9 inet manual
ovs\_bridge br2
ovs\_type OVSPort
  
allow-br2 br2-patch0
iface br2-patch0 inet manual
ovs\_bridge br2
ovs\_type OVSPatchPort
ovs\_patch\_peer br1-patch1 

Redémarrez la VM pour appliquer la configuration :

\[switch@ovs:~$\] sudo reboot

et contrôlez ensuite le statut du service réseau :

\[switch@ovs:~$\] sudo systemctl status networking

### 5 - Test de bon fonctionnement du switch virtuel

Vérifiez à l'aide de la Cde ping la conformité des résultats avec ceux indiqués sur la [maquette](/wp-content/uploads/2018/05/maquette-base-ipfire.png) réseau local virtuel.

Stoppez ensuite la VM ovs support d'Open vSwitch et assurez-vous que les 2 clients Debian ne peuvent plus communiquer entre eux.

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Voilà, c'est terminé !  
Le réseau virtuel de base est créé.  
Le mémento 6.1 vous attend pour  
découvrir le contrôle à distance.

[Mémento 6.1](/acces-locaux-distants/)
