---
title: "Clients - VirtualBox / Debian 10"
date: "2022-04-27"
categories: 
  - "archives"
---

## Mémento 4.1 - Client vm1 cloné vm2

A présent, vous allez compléter le LAN virtuel avec 2 clients Linux reposant également sur Debian 10.

La différence avec les serveurs viendra du bureau Mate, plus simple à installer qu'Openbox et plus complet.

Ceci passera par la création d'un premier client que vous clonerez ensuite pour en générer un second dont vous modifierez uniquement le nom d'hôte et l'adresse IP.

### 1 - Construction du client Linux depuis VirtualBox

L'utilisation de VirtualBox est considérée acquise.

#### _1.1 - Création et configuration de la VM_

Téléchargez l'ISO 32 bits debian-10.x.y-i386-netinst.iso :  
[https://cdimage.debian.org/.../10.10.0/i386/iso-cd/](https://cdimage.debian.org/cdimage/archive/10.10.0/i386/iso-cd/)

Démarrez l'application VirtualBox, puis :

Menu Nouvelle  
\- Nom debian10-vm1 - Type Linux - Version Debian (32...  
\- Taille de la mémoire > 1024 Mo  
\- Disque dur > Créer un disque dur virtuel maintenant  
\- Type de fichier de disque dur > VDI  
\- Stockage sur disque dur ... > Dynamiquement alloué  
\- Emplacement du fichier et taille > 12 Go > Créer

La VM est créée dans le panneau gauche de VirtualBox.

Sélectionnez la nouvelle VM, puis :

Menu Configuration  
\- - - Onglet Général  
\> Avancé > Presse-papier partagé  
\> Bidirectionnel  
  
\- - - Onglet Système  
\> Carte mère > Ordre d'amorçage > Décochez Disquette  
\> Carte mère > Fonctions avancées > Cochez IO-APIC  
\> Processeur > 2 CPU et cochez Activer PAE/NX  
  
\- - - Onglet Affichage  
\> Contrôleur graphique > VMSVGA  
  
\- - - Onglet Stockage  
\> Zone Unités de stockage > Sélectionnez Vide  
\> Zone Attributs > Cliquez sur l'icône CD  
\> Sélectionnez Choisissez un fichier de disque ...  
\> Entrez le chemin de l'image ISO Debian > Ouvrir

Facultatif, accès au dossier partagé par le PC hôte.  
\- - - Onglet Dossiers partagés  
\> Cliquez sur l'icône + > Ajouter un dossier partagé  
\> Chemin du dossier > Sélectionnez Autre...  
\> Accédez à votre dossier > Ex : C:\\Partage-Windows  
\> Sélectionner un dossier > OK > OK

Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de Debian 10 (bureau Mate)_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

Menu Démarrer :  
La VM s'exécute, contrôlez l'ISO affichée et démarrez.

Sélectionnez Graphical Install et appliquez ce qui suit :  
\- Language > French  
\- Pays (territoire ou région) > France  
\- Disposition de clavier à utiliser > Français  
\- Nom de machine > debian10-vm1  
\- Domaine > Laissez le champ vide  
\- MDP du super utilisateur root > Votre MDP root  
\- Confirmation du MDP > Votre MDP root  
\- Nom complet du nouvel utilisateur > Ex: client-linux  
\- Identifiant pour le compte utilisateur > client-linux  
\- MDP pour le nouvel utilisateur > Votre MDP client-linux  
\- Confirmation du MDP > Votre MDP client-linux  
\- Méthode de partitionnement > Assisté - utili... entier  
\- Disque à partitionner > Celui proposé de 12 Go  
\- Schéma de partitionnement > Tout ... seule partition  
\- Table des partitions > Terminer le partitionnement ...  
\- Faut-il appliquer les changements … disques ? > Oui

L'installation de base commence :  
\- Faut-il analyser un autre CD ou DVD ? > Non  
\- Pays du miroir de l'archive Debian > France  
\- Miroir de l'archive Debian > ftp.fr.debian.org  
\- Mandataire HTTP (lais...) > Laissez vide

L'installation continue :  
\- Souhaitez-vous participer à l'étude statistique ... > Non  
\- Logiciels à installer  
\> Décochez environnement de bureau Debian  
\> Sélectionnez ... MATE  
\> Décochez serveur d'impression  
\> Conservez utilitaires usuels du système

L'installation se termine :  
\- Installer ... de démarrage GRUB sur le secteur ... > Oui  
\- Périphérique ... programme de démarrage > /dev/sda  
\- Installation terminée > Continuer _(sans retrait du CD)_

La VM redémarre et affiche une fenêtre de connexion :  
\> Entrez client-linux dans le premier champ  
\> Entrez votre MDP client-linux dans le second champ  
\> Se connecter

[![Capture - Premier démarrage Debian Mate](/wp-content/uploads/2019/04/client-mate-premier-demarrage-430x273.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/04/client-mate-premier-demarrage.png)

Premier démarrage de Debian MATE

Découvrez le contenu de base du bureau MATE.

#### _1.3 - Mise à jour du système Linux_

Depuis le bureau MATE :  
Menu Système  
\> Administration > Gestionnaire de paquets Synaptic

Une fenêtre d'authentification s'ouvre :  
\> Entrez votre MDP root > S'authentifier

La fenêtre du gestionnaire de paquets synaptic s'ouvre :  
\> Ferner de la fenêtre contextuelle Présentation rapide  
\> Recharger pour une MAJ de la liste des paquets  
  
Une fois terminé > Cliquez sur le bouton Etat :  
Si une ligne Installés (pouvant être mis à jour) existe  
\> Cliquez sur le bouton Tout mettre à niveau  
\> Validez en cliquant sur Ajouter à la sélection  
\> Cliquez ensuite sur le bouton Appliquer  
\> Poursuivez en cliquant sur le bouton Appliquer  
\> Une fois la MAJ finie, cliquez sur le bouton Fermer  
Sinon ne faites rien

Gardez l'application ouverte.

#### _1.4 - Francisation de quelques applications_

Vous devez ajouter les 3 paquets ci-dessous :  
\> Rechercher pour filtrer, sélectionner et installer  
\- libreoffice-help-fr _(Acceptez les dépendances)_  
\- gimp-help-fr _(idem)_  
\- firefox-esr-l10n-fr _(idem)_

Fermez ensuite l'application.

#### _1.5 - Installation des utilitaires de VirtualBox_

Ils permettront entre autres le copier/coller et l'accès au dossier partagé par le PC hôte.

Depuis le bureau MATE :  
Menu Applications  
\> Outils système > Terminal MATE

Connectez-vous en tant que root :

\[client-linux@debian10-vm1:~$\] su root
Mot de passe : Entrez votre MDP pour root
\[root@debian10-vm1:~#\]

Relevez la version courante du noyau Linux :

\[root@debian10-vm1:~#\] uname -r

Retour = 4.19.0.x-686-pae

et installez en conséquence les paquets suivants :

\[root@debian10-vm1:~#\] apt install dkms build-essential
\[root@debian10-vm1:~#\] apt install linux-headers-4.19.0-x-686-pae

Ensuite depuis le menu VirtualBox de la VM :  
\> Périphériques > Insérer l'image CD des Additions inv...

Si une fenêtre d'invite d'exécution auto s'ouvre > Annuler

Accédez à l'image, installez les utilitaires, rebootez :

\[root@debian10-vm1:~#\] cd /media/cdrom/
\[root@debian10-vm1:~#\] sh VBoxLinuxAdditions.run
\[root@debian10-vm1:~#\] sudo reboot

et ajustez la taille de l'écran depuis le bureau MATE :  
Menu Système  
\> Préférences > Matériel > Affichage

  
Sans fermer la VM, retirez l'ISO du lecteur CD virtuel :  
Menu Configuration de VirtualBox  
\- - - Onglet Stockage  
\> Zone Unités de stockage > VBoxGuestAdditions.iso  
\> Zone Attributs > Cliquez sur l'icône CD  
\> Retirer le disque du lecteur virtuel > OK

C'est fini, MATE est en place, RAM utilisée < 500 Mo :

[![Capture - Bureau Mate de Debian 10](/wp-content/uploads/2019/04/client-linux-bureau-mate-430x270.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/04/client-linux-bureau-mate.png)

Bureau Mate de Debian 10

Vous pouvez à présent stopper la VM :  
Bureau MATE  
Menu Système > Eteindre

Réduisez ensuite la mémoire vive de la VM à 640 Mo.

#### _1.6 - Montage du dossier partagé par l'hôte_

Démarrez la VM.

Ouvrez un terminal et connectez-vous en tant que root :

\[client-linux@debian10-vm1:~$\] su root
Mot de passe : Entrez votre MDP root
\[root@debian10-vm1:~#\]

Le dossier partagé sera monté dans un sous-dossier de /home/client-linux nommé Partages.

Le trait d'union de l'utilisateur client-linux impose sous systemd de prendre des précautions concernant le nom à donner au service de montage du dossier partagé.

Pour savoir quel nom utiliser, entrez cette Cde :

\[root@debian10-vm1:~#\] systemd-escape -p \\
          --suffix=mount "/home/client-linux/Partages/" 

Le caractère \\ indique d'écrire la Cde sur une seule ligne.  
  
Résultat : home-client\\x2dlinux-Partages.mount  
  
Le bloc \\x2d représente le trait d'union de client-linux.

Sachant cela, créez maintenant le fichier de service :

\[root@debian10-vm1:~#\] cd /etc/systemd/system  
\[root@debian10-vm1:~#\] touch home-client\\x2dlinux-Partages.mount 

Editez celui-ci en omettant le caractère \\ dans le nom :

\[root@debian10-vm1:~#\] nano home-clientx2dlinux-Partages.mount 

et entrez le contenu suivant :

\[Unit\]
Description = Montage dossier partagé fourni par VirtualBox

\[Mount\]
What = _Entrez le nom du dossier partagé par le PC hôte_
Where = home-client\\x2dlinux-Partages
Type = vboxsf
Options=rw,uid=client-linux,gid=client-linux

\[Install\]
WantedBy = multi-user.target

Exemple pour What : What = Partage-Thomas

Intégrez le service dans la configuration de systemd :

\[root@debian10-vm1:~#\] systemctl daemon-reload 

et démarrez celui-ci :

\[root@debian10-vm1:~#\] systemctl start home-clientx2dlinux-Partages.mount 

Vérifiez son statut :

\[root@debian10-vm1:~#\] systemctl status home-clientx2dlinux-Partages.mount 

Si nécessaire, touche q pour quitter le résultat affiché.

Autorisez le lancement du service au boot de la VM :

\[root@debian10-vm1:~#\] systemctl enable home-clientx2dlinux-Partages.mount 

Un lien symbolique est créé.

Pour finir, depuis le bureau MATE :  
Menu Applications  
\> Outils système > Gestionnaire de fichiers Caja  
  
Accédez au dossier partagé par le PC hôte dans :  
/home/clientx2dlinux/Partages

#### _**1.7 - Ajout de l'utilisateur dans le groupe sudo**_

Comme sur les serveurs, ajoutez l'utilisateur client-linux dans le groupe sudo :

\[root@debian10-vm1:~#\] sudo adduser client-linux sudo

#### _1.8 - Configuration du réseau (IP fixe)_

Attention, le nom des cartes réseau respecte une nouvelle nomenclature.

Vérifiez le à l'aide de la Cde suivante :

\[root@debian10-vm1:~$\] ip address

Résultat, l'interface eth0 est renommée enp0s3 :

![Capture - Résultat de la Cde ip address](/wp-content/uploads/2019/04/client-debian-cde-ip-address.png)

Résultat de la Cde ip address

La VM qui sera placée à l'intérieur du LAN impose de modifier le mode d'accès réseau de la carte enp0s3.

Sans stopper la VM, menu Configuration de VirtualBox :  
\- - - Onglet Réseau  
\> Carte 1 > Mode d'accès réseau > Réseau interne  
\> Nom > Remplacez intnet par switch\_interne > OK

Configurez à présent une IP fixe sur la carte enp0s3 :  
Bureau MATE, barre du haut  
\> Clic droit sur l'icône Réseau située à droite  
\> Sélectionnez Modifier les connexions...  
  
Une fenêtre Connexions réseau s'ouvre :  
\> Sélectionnez la seule connexion proposée  
\> Cliquez sur l'icône roue dentée située en bas à gauche  
  
Une fenêtre Modification de ... s'ouvre :  
\- - - Onglet Paramètres IPv4  
\> Méthode > Sélectionnez Manuel > Bouton Ajouter  
\> Champ Adresse : Entrez 192.168.3.2  
\> Champ Masque de réseau : Entrez 255.255.255.0  
\> Champ Passerelle : Entrez 192.168.3.1  
\> Serveurs DNS > Entrez l'IP locale de votre Box Internet  
\> Bouton Enregistrer  
  
Fermez ensuite la fenêtre Connexions réseau.

Le service réseau redémarre automatiquement.

A défaut, relancez celui-ci avec la Cde suivante :

\[root@debian10-vm1:~#\] systemctl restart network-manager

et vérifiez le résultat avec la Cde ip address.

Vérifiez également la table de routage avec la Cde ip r :

![Capture - Table de routage de client-linux](/wp-content/uploads/2019/04/client-linux-table-routage.png)

Table de routage de client-linux

Pour terminer, sélectionnez la VM srvlan :  
Menu Configuration de VirtualBox  
\- - - Onglet Réseau  
\> Carte 2 > Mode d'accès réseau = Réseau interne  
\> Nom > Sélectionnez switch\_interne > OK  
  
Les clients Linux seront ainsi raccordés correctement à srvlan au travers des liaisons nommées switch\_interne.

### 2 - Construction du deuxième client Linux

#### _2.1 - Clonage_

Comme indiqué au début du mémento, vous allez commencer par cloner la VM debian10-vm1.

Stoppez la VM, ensuite :  
\> Panneau gauche de VirtualBox > Sélectionnez celle-ci  
\> Effectuez un clic droit sur la sélection > Cloner...

Une fenêtre s'ouvre :  
\- Nom de la nouvelle machine et chemin  
\> Nom > debian10-vm2  
\> Politique d'adresse MAC > Générer de nouvelles...  
\> Bouton Suivant  
  
\- Type de clone  
\> Clone intégral  
\> Bouton Cloner

Démarrez la nouvelle VM une fois le clonage terminé et connectez-vous sur celle-ci avec les login et MDP de l'hôte debian10-vm1.

#### _2.2 - Changement du nom d'hôte hérité_

Ouvrez le terminal MATE et modifiez le nom d'hôte :

\[client-linux@debian10-vm1:~$\] sudo hostnamectl set-hostname debian10-vm2

Editez ensuite le fichier DNS hosts :

\[client-linux@debian10-vm1:~$\] sudo nano /etc/hosts

et remplacez la valeur debian10-vm1 par debian10-vm2.

Un message d'alerte apparaît, n'en tenez pas compte.

Redémarrez la VM :  
Menu Système > Éteindre... > Redémarrer

Connectez-vous, ouvrez un terminal et vérifiez que le prompt montre bien l'hôte debian10-vm2.

Pour finir, observez le résultat de la Cde hostnamectl :

\[client-linux@debian10-vm2:~$\] hostnamectl 

```
Static hostname: debian10-vm2
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 2a13.....03e4e85b0.....6b75b6d06
           Boot ID: 608f.....5b84e0ca1.....05f2efbea
    Virtualization: oracle
  Operating System: Debian GNU/Linux 10 (buster)
            Kernel: Linux 4.19.0-16-686-pae
      Architecture: x86
```

#### _2.3 - Changement de l'adresse IP héritée_

Effectuez les opérations suivantes :  
Bureau MATE, barre du haut  
\> Clic droit sur l'icône Réseau située à droite  
\> Sélectionnez Modifier les connexions...  
  
Une fenêtre Connexions réseau s'ouvre :  
\> Sélectionnez la seule connexion proposée  
\> Cliquez sur l'icône roue dentée située en bas à gauche  
  
Une fenêtre Modification de ... s'ouvre :  
\- - - Onglet Paramètres IPv4  
\> Champ Adresse : Entrez 192.168.3.4  
\> Bouton Enregistrer  
  
Fermez ensuite la fenêtre Connexions réseau.

Relancez le service réseau :

\[client-linux@debian10-vm2:~$\] sudo systemctl restart network-manager 

et vérifiez le résultat avec la Cde ip address.

_Nota : Si problème d'IP, rebootez la VM et revérifiez._

### 3 - Récapitulatif et test de la maquette réseau

#### _3.1 - Récapitulatif_

Voilà, le réseau virtuel comprend déjà :  
\- 1 PC _hôte_  
\- 3 serveurs virtuels : srvlan, srvsec et srvdmz  
\- 2 clients virtuels : debian10-vm1 et debian10-vm2  
  
\- Le switch virtuel Open vSwitch reste à construire

Retrouvez ceux-ci sur la maquette finale :

[![Synoptique - Maquette du réseau local virtuel avec IPFire](/wp-content/uploads/2018/05/maquette-base-ipfire-430x301.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2018/05/maquette-base-ipfire.png)

  
Configuration de base - Présentation des flux ICMP (ping)

#### _3.2 - Préparation du test_

Le test impose des ajustements sur le PC Hôte et IPFire.

\- PC hôte  
Ajout de 3 routes statiques à la table de routage.

Les Cdes ci-dessous concernent Windows, celles-ci sont différentes sous OS X ou Linux.

Exécutez à présent l'Invite de commande Windows en tant qu'administrateur et observez le contenu de la table de routage courante comme ci-dessous :

\[C:\\~\] route print

Ajoutez ensuite les 3 nouvelles routes comme suit :

\[C:\\~\] route add -p 192.168.2.0 mask 255.255.255.0 192.168.x.w

\[C:\\~\] route add -p 192.168.3.0 mask 255.255.255.0 192.168.x.w

\[C:\\~\] route add -p 192.168.4.0 mask 255.255.255.0 192.168.x.w

192.168.x.w = IP de la carte réseau RED d'IPFire.

Vérifiez-en la prise en compte en réaffichant la table :

\[C:\\~\] route print 

Ces routes permettront entre autres le ping depuis le PC hôte vers les VM du réseau local virtuel.

\- IPFire  
a) Création d'une route statique :  
\> Accédez à IPFire depuis le navigateur Web de srvlan  
\> Menu Réseau > Routes statiques  
\> Champ Adresse IP de l'hôte ... > Entrez 192.168.3.0/24  
\> Champ Passerelle > Entrez 192.168.2.2  
\> Champ Remarque > Route vers VM en 192.168.3.x  
\> Ajouter

Cette route permettra le ping depuis IPFire vers les VM du réseau 192.168.3.0.

[![Capture - Ajout d'une route statique sur IPFire](/wp-content/uploads/2019/01/ipfire_route_statique-430x249.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/01/ipfire_route_statique.png)

Ajout d'une route statique sur IPFire

b) Création d'une règle de pare-feu :  
\> Menu IPFire > Packfire  
\> Installez le module nano _(éditeur de textes)_

Puis éditez depuis la VM IPFire le fichier firewall.local :

\[root@srvsec ~#\] cd /etc/sysconfig  
\[root@srvsec ~#\] nano firewall.local

et ajoutez cette règle sous \## add your 'start' rules here :

iptables -I POLICYFWD -i green0 -s 192.168.3.0/24 -j ACCEPT

Ajoutez-la aussi sous \## add your 'stop' rules here.

Redémarrez IPFire :

\[root@srvsec ~#\] reboot

Cette règle permettra aux VM du réseau 192.168.3.0 d'accéder à Internet.

![Capture - Ajout d'une règle de pare-feu iptables sur IPFire](/wp-content/uploads/2019/02/ipfire_regle_pare_feu.png)

Ajout d'une règle de pare-feu iptables sur IPFire

#### _3.3 - Contrôle du paramétrage serveur DNS_

Vérifiez dans les fichiers /etc/resolv.conf des VM sauf IPFire la présence de :

nameserver 192.16...     _(adresse IP de votre box Internet)_

A défaut, corrigez ou ajoutez cette ligne manuellement.

#### _3.4 - Test du réseau local virtuel (Cde ping)_

Effectuez à présent le test de bon fonctionnement du réseau à l'aide de la Cde ping, ceci en vérifiant la conformité des résultats avec ceux indiqués sur la [maquette](/wp-content/uploads/2018/05/maquette-base-ipfire.png) réseau local virtuel.  
  
Tout doit être correct pour pouvoir continuer.

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Bravo pour être arrivé jusqu'ici !  
Le mémento 5.1 vous attend pour  
l'intégration d'un switch virtuel  
Open vSwitch.

[Mémento 5.1](/open-vswitch-installation-depuis-virtualbox/)
