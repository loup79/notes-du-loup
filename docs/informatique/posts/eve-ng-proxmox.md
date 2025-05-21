---
title: EVE-NG - Proxmox
summary: Installation sur une base Proxmox 8.x.
authors: 
  - G.Leloup
date: 2025-04-21
categories: 
  - EVE-NG
---

## Installation EVE-NG sur Proxmox

Le serveur Proxmox 8.x est considéré opérationnel.

Ressources à réserver pour une VM supportant EVE-NG :  
-\ 8 ou 16 coeurs logiques vCPU  
-\ 8 ou 16 Go de RAM  
-\ Un espace de stockage sur SSD d'au moins 100 Go

Ceci permettra une exploitation confortable.

### Doc / Image ISO / Pack client

a) Doc  
Ouvrez la page suivante :  [Virtual Machine EVE-NG](https://www.eve-ng.net/index.php/documentation/installation/virtual-machine-install/){ target="_blank" }.

Cliquez sur le lien de la version _EVE Community Cookbook_.

La documentation retournée est complète et téléchargeable.

[Fichier PDF](../medias/EVE-CE-BOOK-6.2-2024.pdf){ target="_blank" }.

b) Image ISO  
Téléchargez l'ISO d'EVE-NG à l'aide du lien suivant :  
[Free EVE Community Edition Version 6...](https://www.eve-ng.net/index.php/download/#DL-COMM){ target="_blank" }

c) Pack client  
Téléchargez le pack client pour [Windows](https://mega.nz/file/G5liXYzK#oaSC1Jrh5m0HaNkReirurtrXhIHGw6NOZX3jgus1xqo){ target="_blank" }.

Le pack client est nécessaire pour pouvoir interagir avec les machines sous EVE-NG, il permet d'utiliser des outils comme _wireshark_, _ultravnc_, _putty_, etc...

Il existe également des packs pour Apple OSX et Linux.

### Virtualisation imbriquée

La VM EVE-NG gérera des VM ce qui nécessite une virtualisation imbriquée _(VMs dans VM EVE-NG)_.

Vous devez vérifier que la virtualisation imbriquée est activée sur le serveur Proxmox.

Procédez ainsi pour les processeurs Intel :

```bash
cat /sys/module/kvm_intel/parameters/nested
```

et ainsi pour les processeurs AMD :

```bash
cat /sys/module/kvm_amd/parameters/nested
```

Si le retour = ==Y== ou ==1==, la virtualisation imbriquée est activée.

Sinon, l'activer comme ceci pour un processeur Intel :

```bash
echo "options kvm-intel nested=Y" > /etc/modprobe.d/kvm-intel.conf
```

ou comme ceci pour un processeur AMD :

```bash
echo "options kvm-amd nested=1" > /etc/modprobe.d/kvm-amd.conf
```

Ensuite, redémarrez le serveur Proxmox ou rechargez le module du noyau.

Exemple de rechargement du module pour un processeur Intel :

```bash
modprobe -r kvm_intel
modprobe kvm_intel
```

### Création de la VM EVE-NG

Transférez l'image ISO téléchargée ci-dessus dans le dossier local du serveur Proxmox via son interface web.

<!-- more -->

Ensuite, depuis Proxmox, créez une VM comme ceci :  
\- Onglet Système d'exploitation  
Stockage = local  
Image ISO = Sélectionnez l'ISO téléchargée  
Type = Linux  
Version = 6.x - 2.6 Kernel

\- Onglet Système  
Carte graphique = Par défaut  
Machine = Par défaut (i440fx)  
BIOS = Par défaut (SeaBIOS)  
Contrôleur SCSI = VirtlO SCSI single

\- Onglet Disques  
Bus/périphérique = VirtlO Block  
Stockage = local-lvm  
Taille du disque (Gio) = 128  
Cache = Par défaut (Aucun cache)

\- Onglet Processeur  
Supports de processeur = 1  
Coeurs = 8  
Type = host

\- Onglet Mémoire  
Mémoire (MiB) = 16384

\- Onglet Réseau  
Pont (bridge) = vmbr0  
Modèle = VirtiO (paravirtualisé)

\- Onglet Confirmation  
Vérifiez la configuration et modifiez la si nécessaire.

Cliquez ensuite sur le bouton _Terminer_ pour créer la VM.

### Installation d'EVE-NG sur la VM

Depuis l'interface web de Proxmox, démarrez la VM et suivez les instructions d'installation.

Des tutoriels, nombreux sur Internet, peuvent vous aider si vous rencontrez un problème.

Une fois terminé, effectuez une mise à jour du système :

```bash
# apt update
# apt upgrade
```

Rebootez et vérifiez que le système est bien à jour.

```bash
# apt update
```

Pour info, EVE-NG utilise le service systemd-resolved pour gérer la résolution DNS. Vous pouvez vérifier l'IP du serveur DNS dans le fichier /etc/systemd/resolved.conf.

Ensuite si vous le souhaitez _(Optionnel)_ :  
Créez un utilisateur, ajoutez le au groupe _sudo_ et rebootez.  
Installez le bureau _xfce4_, le gestionnaire de connexion _lightdm_ et un serveur _xrdp_ pour les connexions distantes.

Un serveur ssh est normalement déjà activé sur le port 3389.

L'accès à l'interface web d'EVE-NG se fait ainsi :  
`http://ip-vm-eve-ng`

Les identifiants par défaut pour se connecter sont :

* Nom d'utilisateur : admin
* Mot de passe : eve

Une fois l'interface web ouverte, allez dans :  
-> Management -> User management

Créez un nouveau compte utilisateur déclaré avec le rôle _Administrator_ et supprimez le compte _admin_.

![Image - Rédacteur satisfait](../images/2024/09/redacteur_satisfait_bis.png "Image Pixabay - Mohamed Hassan"){ align=left }

&nbsp;  
Si OK, on peut passer à la suite.  
La partie 2 traite de la création  
d'un labo basique permettant de  
découvrir l'exploitation d'EVE-NG.

[Partie 2](../posts/eve-ng-labo-base.md){ .md-button .md-button--primary }
