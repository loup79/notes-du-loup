---
title: EVE-NG - Labo Simulation Réseau
summary: Ajout d'une VM dans un labo EVE-NG.
authors: 
  - G.Leloup
date: 2024-03-24
categories: 
  - EVE-NG
---

## Ajout d'une VM Debian LXQt

**Mars 2024.**

EVE-NG est installé sur une base Proxmox 8.x.  
[Documentation](../medias/EVE-NG-CE-BOOK-5.5-2024.pdf){ target="_blank" } EVE-NG

### Préparation

Se rendre dans le dossier prévu pour les images QEMU :  

```bash
cd /opt/unetlab/addons/qemu/
```

et créer le dossier qui recevra la VM :

```bash
sudo mkdir linux-nom
```

Transférer ensuite l'image ISO de la distribution Debian Linux dans ce dossier à l'aide de FileZilla en protocole SFTP.

Pour l'authentification SFTP, utiliser l'utilisateur _root_ d'EVE-NG + le MDP de _root_.

Une fois l'image ISO transférée, renommer celle-ci :

<!-- more -->

```bash
sudo mv nom-iso cdrom.iso
```

et la convertir en image QEMU _(Ext .qcow2)_ comme suit :

```bash
sudo /opt/qemu/bin/qemu-img create -f qcow2 virtioa.qcow2 12G
```

12G = Taille souhaitée du disque dur.

Puis ouvrir, selon le processeur supportant EVE-NG, le template associé _linux.yml_ :

```bash
sudo nano /opt/unetlab/html/templates/amd ou intel/linux.yml
```

et compléter la ligne _qemu_options_ avec le paramètre :

```markdown
-k fr
```

La VM disposera ainsi au démarrage d'un clavier français.

### Ajout de la VM dans un labo

Menu Add an object -> _Node_  
\- Sélectionner le template _Linux_  
\- Sélectionner l'image nommée _linux-nom_  
\- Configurer les paramètres _CPU_ _RAM_ et _Ethernets_  
\- Cliquer sur le bouton _Save_

Il suffit ensuite d'un clic droit sur l'icône du noeud + _Start_ pour démarrer la VM et procéder à son installation et sa configuration.

Pour finir, relier le noeud au Network _pnet1 (Cloud1)_ et non au Network _Management(Cloud0)_, l'exploitation du _Cloud0_ posant aujourd'hui un problème sous Proxmox.

### Validation de la VM

Relever l'ID du labo courant (Menu de gauche -> Lab details) et l'ID du noeud créé  (Menu de l'icône -> Edit).

Se rendre ensuite dans le dossier _ID-du-labo/ID-du-noeud/_, exemple :

```bash
cd /opt/unetlab/tmp/0/02d314a2-8ab2-43...../2/
```

_0_ est l'identifiant du créateur du labo courant.

Puis, lancer la Cde suivante :

```bash
sudo /opt/qemu/bin/qemu-img commit virtioa.qcow2
```

L'image QEMU du dossier _linux-nom_ est alors remplacée par celle de la VM installée/configurée.

Le fichier ISO initial peut maintenant être supprimé :

```bash
rm -f /opt/unetlab/addons/qemu/linux-nom/cdrom.iso
```

### Noms d'hôtes et utilisateurs

Si l'on crée un autre noeud à partir de la même image, celui-ci héritera des paramètres de celle-ci, notamment le nom d'hôte et le nom d'utilisateur principal.

Sur une Debian, procéder comme suit pour modifier ces 2 paramètres :

\- Hôte

```bash
sudo hostnamectl set-hostname nouveau-nom
sudo nano /etc/hosts
```

Remplacer dans _hosts_ l'ancienne valeur par _nouveau-nom_.

Pour finir, redémarrer la VM et relancer la Cde _hostnamectl_ pour vérifier la prise en compte de la modification.

\- Utilisateur principal

En tant que _root_ depuis le bureau LXQt, ouvrir une console avec la combinaison de touches _CTRL+ALT+F2_ et entrer les 2 Cdes suivantes :

```bash
usermod -d /home/le-nouveau-nom -m -l le-nouveau-nom ancien-nom
passwd le-nouveau-nom
```

Puis, stopper le noeud depuis l'interface Web du labo, démarrer à nouveau celui-ci et se connecter sur la VM à l'aide du nouvel utilisateur _le-nouveau-nom_ et de son MDP.

### Configuration réseau

Le paramétrage réseau de la VM sous LXQt s'effectue à l'aide de l'outil graphique Connman _(Service connman)_.

Les fichiers de configuration des interfaces réseau se trouvent dans le dossier _/var/lib/connman/_.

Retirer éventuellement une mauvaise configuration réseau dans le fichier _/etc/network/interfaces ( service networking )_.

Vérifier également la configuration du fichier _/etc/hosts_.

### Taille du disque virtuel

Il est possible de redimensionner la taille du disque virtuel de la VM à l'aide de la Cde suivante, ceci VM stoppée :

```bash
cd /opt/unetlab/tmp/0/02d314a2-8ab2-43...../2/
sudo qemu-img resize virtioa.qcow2 +4G
```

+4G signifie que l'on demande une extension de 4 Go.

Redémarrer la VM, installer l'outil graphique _gparted_ sur celle-ci et redimensionner les partitions du disque virtuel afin d'exploiter pleinement la nouvelle taille.

**Fin.**
