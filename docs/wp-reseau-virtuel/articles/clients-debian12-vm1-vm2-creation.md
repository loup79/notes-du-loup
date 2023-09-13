---
title: "vm1 et 2 - VBox/Deb12"
author: G.Leloup
date: "2023-09-12"
categories: 
  - "clients-debian"
---

[![Capture - Debian 12 : Bureau Xfce personnalisé](../wp-content/uploads/2023/09/client-deb12-bureau-xfce-bis-430x210.webp#center "Cliquez pour agrandir l'image")](../wp-content/uploads/2023/09/client-deb12-bureau-xfce-bis.webp)

## Mémento 4.1 - Client vm1 cloné vm2

A présent, vous allez compléter le réseau virtuel avec 2 clients Linux reposant sur Debian 12.
  
L'environnement graphique proposé sera le même que pour srvlan et srvdmz soit le bureau Xfce.  
  
Le premier client sera cloné pour générer le second dont vous ne modifierez que le nom d'hôte et l'adresse IP.

### 1 - Construction du premier client Linux

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
-> Nom : debian12-vm1  
-> Folder : Sélectionnez le dossier de stockage des VM  
-> ISO Image : Sélectionnez l'ISO téléchargée ci-dessus  
-> Type : Linux  
-> Version : Debian (64-bit)  
-> Cochez Skip Unattended Installation (important)  
-> Bouton Suivant
  
-> Mémoire vive : 1024 MB  
-> Processors : 2 CPU si possible  
-> Bouton Suivant

-> Create a Virtual Hard Disk Now : Ajustez à 12 Go  
-> Bouton Suivant

-> Vérifiez le Récapitulatif  
-> Bouton Finish

La VM s'affiche dans le panneau gauche de VirtualBox.

Sélectionnez maintenant la nouvelle VM, puis :  
\- - Menu de VirtualBox > Machine > Configuration...  
\- - - Onglet Général  
-> Avancé > Presse-papier partagé > Bidirectionnel

\- - - Onglet Système  
-> Carte mère > Ordre d'amorçage > Décochez Disquette  
-> Processeur > Cochez Activer PAE/NX

Facultatif, accès au dossier partagé par le PC hôte :  
\- - - Onglet Dossiers partagés  
-> Cliquez sur l'icône + (Ajoute un ... dossier partagé.)  
-> Chemin du dossier > Sélectionnez Autre...  
-> Accédez à votre dossier > Ex : C:\Partage-Windows  
-> Sélectionner un dossier ou Ouvrir > OK > OK
  
Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de la distribution Debian_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

\- - Menu de VirtualBox  > Machine > Démarrer  
-> Démarrage normal _(La VM s'exécute)_  
  
Sélectionnez Graphical Install et appliquez ce qui suit :  
\- Language > Français  
\- Pays (territoire ou région) > France  
\- Disposition de clavier à utiliser > Français  
\- Nom de machine > debian12-vm1  
\- Domaine > Laissez le champ vide  
\- MDP du super utilisateur root > MDP pour root  
\- Confirmation du MDP > MDP pour root  
\- Nom complet du nouvel utilisateur > Ex: client-linux  
\- Identifiant pour le compte utilisateur > client-linux  
\- MDP pour le nouvel utilisateur > MDP pour client-linux  
\- Confirmation du MDP > MDP pour client-linux  
\- Méthode de partitionnement > Assisté - utili... entier  
\- Disque à partitionner > Celui proposé de 12 Go  
\- Schéma de partitionnement > Tout ... seule partition  
\- Table des partitions > Terminer le partitionnement ...  
\- Faut-il appliquer les changements … disques ? > Oui  
  
L'installation commence :  
\- Faut-il analyser d'autres supports ... ? > Non  
\- Pays du miroir de l'archive Debian > France  
\- Miroir de l'archive Debian > deb.debian.org  
\- Mandataire HTTP (lais...) > Laissez vide  
  
L'installation continue :  
\- Souhaitez-vous participer à l'étude statistique ... > Non  
\- Logiciels à installer  
-> Décochez environnement de bureau Debian  
-> Décochez ... GNOME  
-> Sélectionnez ... Xfce  
-> Conservez utilitaires usuels du système  
  
L'installation se termine :  
\- Installer ... de démarrage GRUB sur le secteur ... > Oui  
\- Périphérique ... programme de démarrage > /dev/sda  
\- Installation terminée > Continuer _(sans retrait du CD)_  
  
Le système reboot et une fenêtre de connexion s'ouvre :

[![Capture - Fenêtre de connexion Xfce](../wp-content/uploads/2023/09/client-deb12-login-430x361.webp#center "Cliquez pour agrandir l'image")](../wp-content/uploads/2023/09/client-deb12-login.webp#center)
_**Fenêtre de connexion Xfce**_

-> Premier champ : client-linux  
-> Second champ : MDP de l'utilisateur client-linux  
-> Bouton Se connecter  
  
Le bureau Xfce s'ouvre :

[![Capture - Bureau Xfce](../wp-content/uploads/2023/09/client-deb12-bureau-xfce-430x362.webp#center "Cliquez pour agrandir l'image")](../wp-content/uploads/2023/09/client-deb12-bureau-xfce.webp#center)
_**Bureau Xfce**_

Ouvrez le terminal de Cdes en cliquant sur son icône située en bas à l'intérieur du dock.  
  
Autorisez l'usage de sudo à l'utilisateur client-linux :

```bash
[client-linux@debian12-vm1:~$] su root
Mot de passe : Votre MDP root
[root@debian12-vm1:~#] sudo usermod -aG sudo client-linux
[root@debian12-vm1:~#] exit
```

et redémarrez la VM :  
\- - Menu Applications de Xfce situé en haut à gauche  
-> Déconnexion _(Une fenêtre s'ouvre)_  
-> Bouton Redémarrer  
  
Reconnectez-vous ensuite en tant qu'utilisateur client-linux et rouvrez le terminal de Cdes.

#### _1.3 - Installation des utilitaires de VirtualBox_

Ils permettront entre autres le copier/coller et l'accès au dossier partagé par le PC hôte.

Notez par curiosité la version courante du noyau linux :

```bash
[client-linux@debian12-vm1:~$] uname -r
```

Exemple de retour :

```markdown
6.1.0-11-amd64
```

Installez ensuite les 2 paquets Linux suivants :

```bash
[client-linux@debian12-vm1:~$] sudo apt install dkms build-essential
```

Debian installera automatiquement la dépendance linux-headers-6.1.0-11-amd64.  
  
Accédez ensuite au menu VirtualBox de la fenêtre VM :  
-> Périphériques > Insérer l'image CD des Additions inv...  
  
Montez l'image CD, installez les utilitaires et rebootez :

```bash
[client-linux@deb...] sudo mount /dev/cdrom /media/cdrom
[client-linux@deb...] cd /media/cdrom/
[client-linux@deb...] sudo ./VBoxLinuxAdditions.run
[client-linux@deb...] sudo reboot
```

Reconnectez-vous, la fenêtre de la VM peut à présent être redimensionnée avec la souris. Sa nouvelle taille sera enregistrée au sein de debian12-vm1.
  
Le copier/coller entre le PC hôte et debian12-vm1 doit maintenant fonctionner dans les 2 sens.  
  
Sans fermer la VM, retirez l'image CD du lecteur virtuel :  
\- - Menu de VirtualBox > Machine > Configuration...  
\- - - Onglet Stockage  
-> Zone Unités de stockage > Sélectionnez VBoxGuest...  
-> Zone Attributs > Cliquez sur l'icône CD  
-> Retirer le disque du lecteur virtuel > OK

#### _1.4 - Mise en français de LibreOffice_

Installez les paquets suivants :

```bash
[client-linux@debian12...] sudo apt install libreoffice-l10n-fr
[client-linux@debian12...] sudo apt install libreoffice-help-fr
```

La configuration de base est presque terminée :

[![Capture - Debian 12 : Bureau Xfce personnalisé](../wp-content/uploads/2023/09/client-deb12-bureau-xfce-bis-430x210.webp#center "Cliquez pour agrandir l'image")](../wp-content/uploads/2023/09/client-deb12-bureau-xfce-bis.webp#center)
_**Debian 12 : Bureau Xfce personnalisé**_

Pour un même fond d'écran sur la fenêtre de login Lightdm et le bureau Xfce, procédez ainsi :

```bash
[client-linux@deb...] cd /chemin-de-votre-fond-ecran
[client-linux@deb...] sudo cp fond.jpg /usr/share/backgrounds/
[client-linux@deb...] cd /etc/lightdm
[client-linux@deb...] sudo nano lightdm-gtk-greeter.conf
```

Remplacez la ligne #background= par celle-ci :

```bash
background=/usr/share/backgrounds/fond.jpg
```

#### _1.5 - Montage du dossier partagé par le PC hôte_

Le trait d'union de l'utilisateur client-linux impose sous systemd de prendre des précautions concernant le nom à donner au service de montage du dossier partagé.  
  
Pour savoir quel nom utiliser, entrez cette Cde :

```bash
[client-linux@debian12-vm1:~#] sudo systemd-escape -p \
--suffix=mount "/home/client-linux/Partage/" 
```

Le caractère \ indique d'écrire la Cde sur une seule ligne.

Retour :

```markdown
home-client\x2dlinux-Partage.mount
```

Le bloc \x2d représente le trait d'union de client-linux.  
  
Sachant cela, créez maintenant le fichier de service :

```bash
[client-linux@debian12-vm1:~#] cd /etc/systemd/system

[client-linux@debian12-vm1:~#] sudo touch home-client\x2dlinux-Partage.mount
```

Editez celui-ci en omettant le caractère \ dans le nom :

```bash
[client-linux@debian12-vm1:~#] sudo nano home-clientx2dlinux-Partage.mount 
```

et entrez le contenu suivant :

```bash
[Unit]
Description = Montage dossier partagé fourni par VirtualBox

[Mount]
What = _Entrez le nom du dossier partagé par le PC hôte_
Where = home-client\x2dlinux-Partage
Type = vboxsf
Options=rw,uid=client-linux,gid=client-linux

[Install]
WantedBy = multi-user.target
```

Exemple pour What : What = Partage-Alfred  
  
Intégrez le service dans la configuration de systemd :

```bash
[client-linux@debian12-vm1:~#] sudo systemctl daemon-reload 
```

et démarrez celui-ci :

```bash
[client-linux@debian12-vm1:~#] sudo systemctl start home-clientx2dlinux-Partage.mount 
```

Vérifiez ensuite son statut :

```bash
[client-linux@debian12-vm1:~#] sudo systemctl status home-clientx2dlinux-Partage.mount 
```

Touche q pour quitter le résultat affiché.
  
Si statut = active, autorisez le service au boot de la VM :

```bash
[client-linux@debian12-vm1:~#] sudo systemctl enable home-clientx2dlinux-Partage.mount 
```

Un lien symbolique vers le service est créé.  
  
Pour finir, ouvrez le gestionnaire de fichiers :  
\- - Menu Applications de Xfce situé en haut à gauche  
-> Système > Gestionnaire de fichiers Thunar  
  
et observez le contenu partagé par le PC hôte dans :  
/home/clientx2dlinux/Partage

#### _1.6 - Configuration du réseau (IP fixe)_

Avant, vérifiez l'IP courante avec la Cde ip address :

```bash
[client-linux@debian12-vm1:~$] ip address
```

Résultat, IP 10.0.2.15, IP fournie par VirtualBox :

![Capture - Résultat de la Cde ip address](../wp-content/uploads/2023/09/client-deb12-ip-address-580x317.webp#center)
_**Résultat de la Cde ip address**_

La VM debian12-vm1 étant prévue en zone LAN, il faut changer le mode d'accès réseau de sa carte enp0s3.
  
Pour cela, sélectionnez la VM dans VirtualBox :  
\- - Menu de VirtualBox > Machine > Configuration...  
\- - - Onglet Réseau  
-> Adapter 1 > Mode d'accès réseau > Réseau interne  
-> Name > Remplacez intnet par switch_interne > OK  
  
Configurez à présent une IP fixe sur la carte enp0s3 :  
\- - Bureau Xfce, barre du haut  
-> Clic droit sur l'icône Réseau située à droite  
-> Sélectionnez Modifier les connexions...  
  
Une fenêtre Connexions réseau s'ouvre :  
-> Sélectionnez la connexion Wired connection 1  
-> Cliquez sur l'icône roue dentée de la fenêtre
  
Une fenêtre Modification de ... s'ouvre :  
-> Nom de la connexion > Entrez Connexion carte 1  
  
\- - - Onglet Ethernet  
-> Périphérique > Sélectionnez enp0s3  
  
\- - - Onglet Paramètres IPv4  
-> Méthode > Sélectionnez Manuel > Bouton Ajouter  
-> Champ Adresse : Entrez 192.168.3.2  
-> Champ Masque de réseau : Entrez 255.255.255.0  
-> Champ Passerelle : Entrez 192.168.3.1  
-> Serveurs DNS > Entrez l'IP locale de votre Box Internet  
-> Bouton Enregistrer  
  
Fermez ensuite la fenêtre Connexions réseau.  
Le service réseau redémarre automatiquement.  
  
Vérifiez par prudence la bonne configuration du réseau :

```bash
[client-linux@debian12-vm1:~#] ip address
[client-linux@debian12-vm1:~#] nmcli  # Cde NetworkManager
```

![Capture - Résultat de la Cde nmcli](../wp-content/uploads/2023/09/client-deb12-cde-nmcli-580x551.webp#center)
_**Résultat de la Cde nmcli**_

Les fichiers configurés avec NetworkManager sont ici :  
/etc/NetworkManager/system-connections/  
  
Le service réseau peut être redémarré avec cette Cde :

```bash
[client-linux@debian12-vm1:~#] sudo systemctl restart NetworkManager
```

Par curiosité, lisez la table de routage avec la Cde ip r :

![Capture - Table de routage de debian12-vm1 ](../wp-content/uploads/2023/09/client-deb12-table-routage-580x66.webp#center)
_**Table de routage de debian12-vm1**_

Pour finir, sélectionnez la VM srvlan dans VirtualBox :  
\- - Menu Configuration de VirtualBox > Machine > Configuration...  
\- - - Onglet Réseau  
-> Adapter 2 > Mode d'accès réseau = Réseau interne  
-> Name > Remplacez intnet par switch_interne > OK  
  
Les clients Linux seront ainsi raccordés correctement à srvlan au travers des liaisons nommées switch_interne.  
  
_Nota : La VM debian12-vm1 accède à Internet si les VM srvsec et srvlan sont démarrées._

### 2 - Construction du deuxième client Linux

#### _2.1 - Clonage_

Comme indiqué au début du mémento, vous allez commencer par cloner la VM debian12-vm1.

Stoppez la VM, ensuite :  
-> Panneau gauche de VirtualBox > Sélectionnez celle-ci  
-> Effectuez un clic droit sur la sélection > Cloner...  
  
Une fenêtre Cloner la machine virtuelle s'ouvre :  
\- Nom de la nouvelle machine et chemin  
-> Name > Entrez debian12-vm2  
-> MAC Address Policy > Générer de nouvelles ...  
-> Bouton Suivant  
  
\- Type de clone  
-> Sélectionnez Clone intégral  
-> Bouton Finish

Le clonage s'exécute.
  
Démarrez la nouvelle VM une fois le clonage terminé et connectez-vous sur celle-ci avec les login et MDP de l'hôte debian12-vm1.

#### _2.2 - Changement du nom d'hôte hérité_

Ouvrez le terminal et modifiez le nom d'hôte :

```bash
[client-linux@debian12-vm1:~$] sudo hostnamectl set-hostname debian12-vm2
```

Editez ensuite le fichier DNS hosts :

```bash
[client-linux@debian12-vm1:~$] sudo nano /etc/hosts
```

et remplacez debian12-vm1 par debian12-vm2.
  
Un message d'alerte apparaîtra, n'en tenez pas compte.  
  
Redémarrez la VM, connectez-vous avec les mêmes login et MDP que précédemment et ouvrez le terminal pour vérifier que le prompt montre bien debian12-vm2.
  
Pour finir, observez le résultat de la Cde hostnamectl :

```bash
[client-linux@debian11-vm2:~$] hostnamectl 
```

Retour :

```markdown
Static hostname: debian12-vm2
       Icon name: computer-vm
         Chassis: vm
      Machine ID: ea1ed91bc3854862b45b2c3f80c009e6
         Boot ID: 9e8fc772ba124dcca6cf7c56466fe4b5
  Virtualization: oracle
Operating System: Debian GNU/Linux 12 (bookworm)  
          Kernel: Linux 6.1.0-11-amd64
    Architecture: x86-64
```

#### _2.3 - Changement de l'adresse IP héritée_

Effectuez les opérations suivantes :  
\- - Bureau Xfce, barre du haut  
-> Clic droit sur l'icône Réseau située à droite  
-> Sélectionnez Modifier les connexions...  
  
Une fenêtre Connexions réseau s'ouvre :  
Supprimez la ou les connexions affichées en cliquant sur l'icône - située en bas de la fenêtre.  
  
Ensuite, cliquez sur l'icône + pour créer une connexion.  
  
Une fenêtre Sélectionner un type de connexion s'ouvre :  
-> Sélectionnez Ethernet > Bouton Créer...  
  
Une fenêtre Modification de ... s'ouvre :  
-> Nom de la connexion > Connexion carte 1  
  
\- - - Onglet Ethernet  
-> Périphérique > Sélectionnez enp0s3  
  
\- - - Onglet Paramètres IPv4  
-> Méthode > Sélectionnez Manuel > Bouton Ajouter  
-> Champ Adresse : Entrez 192.168.3.4  
-> Champ Masque de réseau : Entrez 255.255.255.0  
-> Champ Passerelle : Entrez 192.168.3.1  
-> Serveurs DNS > Entrez l'IP locale de votre Box Internet  
-> Bouton Enregistrer  
  
Fermez ensuite la fenêtre Connexions réseau.  
  
Le service réseau redémarre automatiquement, sinon :

```bash
[client-linux@debian12-vm2:~$] sudo systemctl restart NetworkManager 
```

Vérifiez la configuration avec la Cde ip address.  
  
_Nota : Si problème d'IP, rebootez la VM et revérifiez._

### 3 - Récapitulatif et test de la maquette réseau

#### _3.1 - Récapitulatif_

Voilà, le réseau virtuel comprend déjà :  
\- 1 PC hôte _(Windows ou Linux)_  
\- 3 serveurs virtuels : srvlan, srvsec et srvdmz  
\- 2 clients virtuels : debian12-vm1 et debian12-vm2  
  
\- Le switch virtuel Open vSwitch reste à construire.  
  
Retrouvez ceux-ci sur la maquette finale :

[![Synoptique - Maquette du réseau local virtuel avec IPFire](../wp-content/uploads/2018/05/maquette-base-ipfire-430x301.png#center "Cliquez pour agrandir l'image")](../wp-content/uploads/2018/05/maquette-base-ipfire.png#center)
_**Configuration de base - Flux ICMP (ping)**_

#### _3.2 - Préparation du test_

Le test impose d'ajouter 3 routes statiques dans la table de routage du PC hôte.  
  
Les Cdes ci-dessous concernent Windows, celles-ci sont différentes sous OS X ou Linux.  
  
Ouvrez à présent le Windows PowerShell (admin) et observez le contenu de la table de routage :

```bash
[C:\~] route print
```

Ajoutez ensuite les 3 nouvelles routes comme suit :

```bash
[C:\~] route add -p 192.168.2.0 mask 255.255.255.0 192.168.x.w

[C:\~] route add -p 192.168.3.0 mask 255.255.255.0 192.168.x.w

[C:\~] route add -p 192.168.4.0 mask 255.255.255.0 192.168.x.w
```

\- Le commutateur \-p rend la route créée permanente.  
\- L'adresse 192.168.x.w = IP de la carte RED d'IPFire.

Vérifiez le résultat en réaffichant la table de routage :

```bash
[C:\~] route print 
```

Ces routes permettront entre autres le ping depuis le PC hôte vers les VM du réseau local virtuel.

Pour supprimer une route caduque, utilisez cette Cde :

```bash
[C:\~] route delete 192.168.x.0 mask 255.255.255.0
```

#### _3.3 - Contrôle du paramétrage serveur DNS_

Hormis IPFire, vérifiez dans les fichiers /etc/resolv.conf des VM la présence de :

```bash
nameserver 192.16...     # adresse IP de votre box Internet
```

A défaut, corrigez ou ajoutez cette ligne manuellement.

#### _3.4 - Test du réseau local virtuel (Cde ping)_

Effectuez à présent le test de bon fonctionnement du réseau à l'aide de la Cde ping, ceci en vérifiant la conformité des résultats avec ceux indiqués sur la [maquette](../wp-content/uploads/2018/05/maquette-base-ipfire.png) réseau local virtuel.  
  
Tout doit être correct pour pouvoir continuer.

![Image - Rédacteur satisfait](../wp-content/uploads/2023/07/redacteur_satisfait.jpg "Image Pixabay - Mohamed Hassan"){ align=left }

&nbsp;  
Bravo pour être arrivé jusqu'ici !  
Le mémento 5.11 vous attend pour  
l'intégration d'un switch virtuel  
Open vSwitch.

[Mémento 5.1](../virtualbox-debian11-openvswitch-creation){ .md-button }
