---
title: "Open vSwitch - VBox/Deb12"
summary: Installation et configuration d'un switch virtuel Open vSwitch.
authors: 
  - G.Leloup
date: 2023-11-03
categories: 
  - 06 OvS + Conteneurs LXC
---

<figure markdown>
  ![Image Pixabay - Switch informatique de Bru-nO](../images/2023/10/openvswitch_3.webp){ width="430" }
</figure>

## Mémento 5.1 - Switch virtuel OvS

Open vSwitch _(OvS)_ se comportera comme un véritable switch physique en proposant la gestion des adresses MAC tel un switch L2 ainsi que celle des adresses IP pour le routage des paquets IP tel un switch L3.
  
Le switch virtuel sera, contrairement à une architecture standard, installé sur une VM et non sur le PC hôte de l'hyperviseur VirtualBox.

Le raccordement externe des clients LAN Debian sur le switch se fera à l'aide de ports patch.

<!-- more -->
  
Le moyen d'approcher l'architecture standard sera de créer un conteneur à l'intérieur de la VM et de connecter celui-ci sur OVS à l'aide d'une interface réseau virtuelle Veth _(Mémento suivant)_.  
  
La gestion possible de VLAN ne sera pas traitée ici.

### Construction de la VM ovs

L'utilisation de VirtualBox est considérée acquise.  
  
A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](../posts/virtualbox-installation.md){ target="_blank" }  
[VirtualBox - Mode d’accès réseau par pont](../posts/virtualbox-pont-reseau.md){ target="_blank" }

#### _- Création et configuration_

Le PC hôte doit être un PC 64 bits, courant de nos jours.  
  
Téléchargez l'ISO debian-12.x.y-amd64\-netinst.iso :  
[https://cdimage.debian.org/.../current/amd64/iso-cd/](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/){ target="_blank"}  
  
\- Démarrez ensuite l'application VirtualBox 7.x, puis :  
\- - Menu de VirtualBox > Machine > Nouvelle...  
-> Nom : ovs  
-> Folder : Sélectionnez le dossier de stockage des VM  
-> ISO Image : Sélectionnez l'ISO téléchargée ci-dessus  
-> Type : Linux  
-> Version : Debian (64-bit)  
-> Cochez Skip Unattended Installation _(important)_  
-> Bouton Suivant  

-> Mémoire vive : 1024 MB  
-> Processors : 2 CPU si possible  
-> Bouton Suivant

-> Create a Virtual Hard Disk Now : Ajustez à 12 Go  
-> Bouton Suivant

-> Vérifiez le Récapitulatif  
-> Bouton Finish
  
La VM s'affiche dans le panneau gauche de VirtualBox.  
  
\- Sélectionnez maintenant la nouvelle VM, puis :  
\- - Menu de VirtualBox > Machine > Configuration...  
\- - - Onglet Général  
-> Avancé > Presse-papier partagé > Bidirectionnel

\- - - Onglet Système  
-> Carte mère > Ordre d'amorçage > Décochez Disquette  
-> Processeur > Cochez Activer PAE/NX
  
-> OK  
  
Les autres paramètres peuvent rester inchangés.

#### _- Installation de Debian 12_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparait lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.  
  
\- - Menu de VirtualBox > Machine > Démarrer  
-> Démarrage normal _(La VM s'exécute)_
  
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
\- Disque à partitionner > Celui proposé de 12 Go  
\- Schéma de partitionnement > Tout ... seule partition  
\- Table des partitions > Terminer le partitionnement ...  
\- Faut-il appliquer les changements … disques ? > Oui  
  
L'installation de base commence :  
\- Faut-il analyser d'autres supports ... ? > Non  
\- Pays du miroir de l'archive Debian > France  
\- Miroir de l'archive Debian > deb.debian.org  
\- Mandataire HTTP (lais...) > Laissez vide  
  
L'installation continue :  
\- Souhaitez-vous participer à l'étude statistique... > Non  
\- Logiciels à installer  
-> Décochez environnement de bureau Debian  
-> Décochez ... GNOME  
-> Conservez utilitaires usuels du système  
  
L'installation se termine :  
\- Installer ... de démarrage GRUB sur le secteur ... > Oui  
\- Périphérique ... programme de démarrage > /dev/sda  
\- Installation terminée > Continuer _(sans retrait du CD)_  
  
Le système reboot et une fenêtre de connexion s'ouvre :  
-> ovs login : Entrez switch  
-> Password : Entrez Votre MDP switch  
Si tout est OK, affichage du prompt switch@ovs:~$

<figure markdown>
  ![Capture - Open vSwitch : Premier démarrage de la VM ovs](../images/2023/10/ovs-deb12-premier-demarrage.webp)
  <figcaption>Open vSwitch : Premier démarrage de la VM ovs</figcaption>
</figure>

Donnez à présent les droits d'administrateur root à l'utilisateur switch :

```bash
[switch@ovs:~$] su root
Mot de passe : Votre MDP root
[root@ovs:~#] apt install sudo
[root@ovs:~#] sudo usermod -aG sudo switch
[root@ovs:~#] sudo reboot
```

et reconnectez-vous en tant qu'utilisateur switch.  
  
Pour info, la VM peut être arrêtée de 2 façons :

```bash
[switch@ovs:~$] sudo poweroff  
[switch@ovs:~$] sudo shutdown -h now
```

### Installation-configuration d'OvS

Au préalable, observez la configuration réseau active :

<figure markdown>
  ![Capture - Open vSwitch : Cartes réseau de base lo et enp0s3](../images/2023/10/ovs-deb12-ip-address.webp)
  <figcaption>Open vSwitch : Cartes réseau de base lo et enp0s3</figcaption>
</figure>

#### _- Installation_

Installez le paquet openvswitch-switch :

```bash
[switch@ovs:~$] sudo apt install openvswitch-switch
```

Les paquets concernant les dépendances manquantes sont ajoutés automatiquement.

Affichez maintenant la version d'Open vSwitch _(OvS)_ :

```bash
[switch@ovs:~$] sudo ovs-vsctl show
```

<figure markdown>
  ![Capture - Open vSwitch : Affichage de la version](../images/2023/10/ovs-deb12-version-openvswitch.webp)
  <figcaption>Open vSwitch : Affichage de la version</figcaption>
</figure>

ainsi que les informations du module openvswitch intégré dans le noyau linux :

```bash
[switch@ovs:~$] sudo modinfo openvswitch
```

<figure markdown>
  ![Capture - Open vSwitch : Informations sur le module noyau openvswitch.ko](../images/2023/10/ovs-deb12-module-noyau-openvswitch.webp)
  <figcaption>Open vSwitch : Chemin du module openvswitch.ko</figcaption>
</figure>

Contrôlez enfin le bon chargement du module :

```bash
[switch@ovs:~$] lsmod | grep openvswitch
```

<figure markdown>
  ![Capture - Open vSwitch : Module openvswitch.ko chargé](../images/2023/10/ovs-deb12-modules-lsmod.webp)
  <figcaption>Open vSwitch : Module openvswitch.ko chargé</figcaption>
</figure>

\- - Liste des [principaux composants](../images/2023/10/openvswitch-composants.webp){:target="_blank"} OVS installés - -
  
\- /usr/sbin/ovs-vswitchd :  
Le service de commutation _(compatible OpenFlow)_.  
  
\- /usr/bin/ovs-appctl :  
Dialogue CLI _(Command Line Interface)_ avec le service.  
  
\- /usr/bin/ovs-vsctl :  
Configuration CLI du service via le serveur de Bdd.  
  
\- /usr/sbin/ovsdb-server :  
Le serveur de Bdd contenant la configuration du service.  
  
\- /usr/bin/ovsdb-client :  
Dialogue CLI avec le serveur de Bdd _(backup, etc...)_.  
  
\- /usr/bin/ovsdb-tool :  
Configuration CLI des fichiers de la Bdd _(création, etc...)_.  
  
\- /usr/bin/ovs-dpctl :  
Configuration CLI des datapaths du noyau d'OVS.  
  
\- /usr/bin/ovs-ofctl :  
Configuration CLI de la partie OpenFlow d'OVS.  
  
La Bdd conf.db se situe dans /etc/openvswitch/.  
Les logs se situent dans /var/log/openvswitch/.

Contrôlez l'activation du service openvswitch-switch :

```bash
[switch@ovs:~$] sudo systemctl status openvswitch-switch
```

<figure markdown>
  ![Capture - Open vSwitch : Service openvswitch-switch activé](../images/2023/10/ovs-deb12-service-openvswitch-switch.webp)
  <figcaption>Open vSwitch : Service openvswitch-switch activé</figcaption>
</figure>

ainsi que celle des 2 services suivants :

```bash
[switch@ovs:~$] sudo systemctl status ovsdb-server
[switch@ovs:~$] sudo systemctl status ovs-vswitchd
```

#### _- Configuration_

Commencez par créer un bridge _(switch)_ de nom br0 :

```bash
[switch@ovs:~$] sudo ovs-vsctl add-br br0   
[switch@ovs:~$] sudo ovs-vsctl show
```

<figure markdown>
  ![Capture - Open vSwitch : Vue création bridge br0](../images/2023/10/ovs-deb12-creation-br0.webp)
  <figcaption>Open vSwitch : Vue création bridge br0</figcaption>
</figure>

Un port et une interface virtuels de même nom ont été associés au bridge.

Utilisez la Cde del-br pour détruire un bridge existant.

Observez les infos détaillées du bridge br0 comme suit :

```bash
[switch@ovs:~$] sudo ovs-vsctl list br br0
```

et la création de l'interface br0 avec la Cde ip address :

<figure markdown>
  ![Capture - Open vSwitch : Vue interface br0](../images/2023/10/ovs-deb12-interface-reseau-br0.webp)
  <figcaption>Open vSwitch : Vue interface br0</figcaption>
</figure>

Testez l'interface enp0s3 avec un ping vers yahoo.fr, celui-ci doit recevoir une réponse positive.

Rattachez à présent l'interface enp0s3 au bridge br0 :

```bash
[switch@ovs:~$] sudo ovs-vsctl add-port br0 enp0s3  
[switch@ovs:~$] sudo ovs-vsctl show
```

<figure markdown>
  ![Capture - Open vSwitch : Vue affectation enp0s3 sur br0](../images/2023/10/ovs-deb12-affectation-port-enp0s3.webp)
  <figcaption>Open vSwitch : Vue affectation enp0s3 sur br0</figcaption>
</figure>

Le port virtuel enp0s3 et l'interface virtuelle enp0s3 ne font qu'un.

Utilisez la Cde del-port pour détruire un port existant.

Observez de nouveau le retour de la Cde ip address :

<figure markdown>
  ![Capture - Open vSwitch : Mêmes adresses MAC enp0s3/br0](../images/2023/10/ovs-deb12-affectation-interface-enp0s3.webp)
  <figcaption>Open vSwitch : Mêmes adresses MAC enp0s3/br0</figcaption>
</figure>

La carte virtuelle br0 possède à présent la même adresse MAC que la carte virtuelle enp0s3.

### Intégration de la VM ovs

#### _- Configuration réseau VBox_

Stoppez la VM :

```bash
[switch@ovs:~$] sudo poweroff
```

Sélectionnez ensuite celle-ci dans VirtualBox, puis :  
\- - Menu de VirtualBox > Machine > Configuration...  
\- - - Onglet Réseau  
-> Adapter 1  
-> Mode d'accès réseau > Réseau interne  
-> Name > Sélectionnez switch_interne  
-> Avancé > Mode Promiscuité  
-> Sélectionnez Allow VMs

-> Adapter 2  
-> Cochez Activer l'interface réseau  
-> Mode d'accès réseau > Réseau interne  
-> Name > Entrez liaison_vm1  
-> Avancé > Mode Promiscuité  
-> Sélectionnez Allow VMs

-> Adapter 3  
-> Cochez Activer l'interface réseau  
-> Mode d'accès réseau > Réseau interne  
-> Name > Entrez liaison_vm2  
-> Avancé > Mode Promiscuité  
-> Sélectionnez Allow VMs

-> OK

Redémarrez la VM.

Il est impossible, OVS n'étant pas installé sur le PC hôte de VirtualBox, d'attribuer aux interfaces 2 et 3 un mode d'accès réseau généralement utilisé avec OVS soit un mode de type Accès par pont au travers d'une interface réseau virtuelle TAP _( [Voir le § 4](#titre-4) )_.
  
C'est donc le type Réseau interne qui sera utilisé.  
  
Le mode promiscuité Allow VMs permettra à la VM ovs de travailler tel un switch et traiter ainsi tout le trafic réseau à destination et en provenance d'autres VM.  
  
Il est nécessaire, dans une infrastructure virtualisée telle Open vSwitch installé sur une VM, d'appliquer le mode promiscuité à une interface réseau devant agir comme un pont _(bridge)_.

#### _- Configuration réseau Debian_

Vous allez à présent configurer OVS au boot de la VM.

Editez pour cela le fichier réseau interfaces :

```bash
[switch@ovs:~$] sudo nano /etc/network/interfaces
```

et modifiez le comme suit :

```bash
# This file describes the network interfaces available on ...
# and how to activate them. For more information, see ...

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
# Mettre un # devant les 2 lignes ci-dessous
#allow-hotplug enp0s3
#iface enp0s3 inet dhcp

## Configuration Open vSwitch
# Activation de l'interface br0
auto br0
allow-ovs br0

# Configuration IP de l'interface br0
iface br0 inet static
address 192.168.3.15
netmask 255.255.255.0
gateway 192.168.3.1
ovs_type OVSBridge
ovs_ports enp0s3

# Attachement du port/interface enp0s3 au bridge br0
allow-br0 enp0s3
iface enp0s3 inet manual
ovs_bridge br0
ovs_type OVSPort
```

Redémarrez ensuite le service networking.service :

```bash
[switch@ovs:~$] sudo systemctl restart networking
```

et contrôlez le résultat avec la Cde ip address :

<figure markdown>
  ![Capture - Open vSwitch : Vue adresse IP sur interface br0](../images/2023/10/ovs-deb12-adresse-ip-br0.webp)
  <figcaption>Open vSwitch : Vue adresse IP sur interface br0</figcaption>
</figure>

Constat :  
\- 2 nouvelles interfaces réseau enp0s8 et enp0s9  
\- L'interface br0 a la même adresse MAC que enp0s3  
\- L'interface br0 possède l'adresse IP 192.168.3.15  
\- Le ping vers l'adresse lP 192.168.3.1 fonctionne

L'adresse IP 192.168.3.15 sera utilisée plus tard pour administrer Open vSwitch à distance.

**Important** : Le bon démarrage du réseau au boot de la VM ovs a exigé l'ajout dans la section Service du fichier /usr/lib/systemd/system/networking.service d'une ligne retardant de 6 secondes l'activation de celui-ci.

```bash
...
[Service]
...
ExecStartPre=/usr/bin/sleep 6  # Ligne ajoutée
ExecStart=/sbin/ifup -a ...
...
```

Suivre les prochaines MAJ de paquets Open vSwitch.

### Raccordement vm1-2 sur OvS {#titre-4}

Open vSwitch est généralement installé sur le PC hôte de l'hyperviseur. Un bridge tel br0 utilise alors les interfaces réseau physiques du PC hôte pour l'accès à Internet et des interfaces réseau virtuelles TAP de niveau L2 pour l'accès aux VM.

Dans le cas présent, Open vSwitch est installé sur une VM de VirtualBox.

Vous allez donc raccorder les 2 clients Debian sur les interfaces réseau enp0s8/9 de la VM ovs, interfaces qui seront rattachées à des bridges de nom br1/2.

Vous lierez les 3 bridges à l'aide de ports patch :

<figure markdown>
  ![Capture - Open vSwitch : Bridges montés en cascade](../images/2023/10/ovs-deb12-patch-bridges.webp)
  <figcaption>Open vSwitch : Bridges montés en cascade</figcaption>
</figure>

L'ensemble des bridges rattachés avec des ports patch (ports de brassage) peut être vu comme un seul pont.  
  
La création de VLAN et le routage de paquets IP entre ceux-ci restent possibles.

#### _- Création des bridges-ports_

Créez les bridges br1 et br2 :

```bash
[switch@ovs:~$] sudo ovs-vsctl add-br br1  
[switch@ovs:~$] sudo ovs-vsctl add-br br2
```

Rattachez les interfaces enp0s8/9 aux bridges br1/2 :

```bash
[switch@ovs:~$] sudo ovs-vsctl add-port br1 enp0s8  
[switch@ovs:~$] sudo ovs-vsctl add-port br2 enp0s9
```

Reliez maintenant br0 avec br1 :

```bash
[switch@ovs:~$] sudo ovs-vsctl -- add-port br0 br0-patch0 \
-- set interface br0-patch0 type=patch options:peer=br1-patch0
 
[switch@ovs:~$] sudo ovs-vsctl -- add-port br1 br1-patch0 \
-- set interface br1-patch0 type=patch options:peer=br0-patch0  
```

Le caractère \ indique d'écrire le tout sur une seule ligne.  

puis br1 avec br2 :

```bash
[switch@ovs:~$] sudo ovs-vsctl -- add-port br1 br1-patch1 \
-- set interface br1-patch1 type=patch options:peer=br2-patch0 
 
[switch@ovs:~$] sudo ovs-vsctl -- add-port br2 br2-patch0 \
-- set interface br2-patch0 type=patch options:peer=br1-patch1
```  

Contrôlez la prise en compte de la configuration :

```bash
[switch@ovs:~$] sudo ovs-vsctl  show
```

<figure markdown>
  ![Capture - Open vSwitch : Contrôle configuration patch](../images/2023/10/ovs-deb12-controle-patch-bridges.webp)
  <figcaption>Open vSwitch : Contrôle configuration patch</figcaption>
</figure>

#### _- Paramétrage réseau Debian_

Editez le fichier interfaces :

```bash
[switch@ovs:~$] sudo nano /etc/network/interfaces
```

et modifiez le comme suit :

```bash
# This file describes the network interfaces available on ...
# and how to activate them. For more information, see ...

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
# Mettre un # devant les 2 lignes ci-dessous
#allow-hotplug enp0s3
#iface enp0s3 inet dhcp

## Configuration Open vSwitch
# Configuration du bridge br0
auto br0
allow-ovs br0

# Configuration IP de l'interface br0
iface br0 inet static
address 192.168.3.15
netmask 255.255.255.0
gateway 192.168.3.1
ovs_type OVSBridge
ovs_ports enp0s3 br0-patch0

# Attachement du port/interface enp0s3 au bridge br0
allow-br0 enp0s3
iface enp0s3 inet manual
ovs_bridge br0
ovs_type OVSPort

# Liaison patch br0 vers br1
allow-br0 br0-patch0
iface br0-patch0 inet manual
ovs_bridge br0
ovs_type OVSPatchPort
ovs_patch_peer br1-patch0  
  
# Configuration du bridge br1
auto br1
allow-ovs br1
iface br1 inet manual
ovs_type OVSBridge
ovs_ports enp0s8 br1-patch0 br1-patch1
  
allow-br1 enp0s8
iface enp0s8 inet manual
ovs_bridge br1
ovs_type OVSPort

# Liaisons patch br1 vers br0 et br2 
allow-br1 br1-patch0
iface br1-patch0 inet manual
ovs_bridge br1
ovs_type OVSPatchPort
ovs_patch\_peer br0-patch0
 
allow-br1 br1-patch1
iface br1-patch1 inet manual
ovs_bridge br1
ovs_type OVSPatchPort
ovs_patch\_peer br2-patch0

# Configuration du bridge br2
auto br2
allow-ovs br2
iface br2 inet manual
ovs_type OVSBridge
ovs_ports enp0s9 br2-patch0
  
allow-br2 enp0s9
iface enp0s9 inet manual
ovs_bridge br2
ovs_type OVSPort

 # Liaison patch br2 vers br1 
allow-br2 br2-patch0
iface br2-patch0 inet manual
ovs_bridge br2
ovs_type OVSPatchPort
ovs_patch\_peer br1-patch1
```

Redémarrez la VM pour appliquer la configuration :

```bash
[switch@ovs:~$] sudo reboot
```

puis contrôlez le statut du service réseau :

```bash
[switch@ovs:~$] sudo systemctl status networking
```

ainsi que le contenu de la Bdd d'Open vSwitch :

```bash
[switch@ovs:~$] sudo ovs-vsctl show
```

#### _- Paramétrage réseau VBox_

Sans stopper les VM, modifiez l'onglet réseau des 2 clients debian12-vm\* comme suit :  
-> Adapter 1  
-> Mode d'accès réseau > Réseau interne  
-> Name > Sélectionnez liaison_vm\* selon la VM  
-> OK

### Test du fonctionnement d'OvS

Vérifiez à l'aide de la Cde ping la conformité des résultats avec ceux indiqués sur la [maquette](../images/2018/05/maquette-base-ipfire.png){ target="_blank" } réseau local virtuel.

Stoppez ensuite la VM ovs support d'Open vSwitch et assurez-vous que les 2 clients Debian ne peuvent plus communiquer entre eux.

![Image - Rédacteur satisfait](../images/2023/07/redacteur_satisfait.jpg "Image Pixabay - Mohamed Hassan"){ align=left }

&nbsp;  
Voilà, c'est terminé !  
Le réseau virtuel de base est créé.  
Le mémento 5.2 vous attend pour  
découvrir les conteneurs.

[Mémento 5.2](../posts/podman-debian12-lxc-partie-1.md){ .md-button .md-button--primary }
