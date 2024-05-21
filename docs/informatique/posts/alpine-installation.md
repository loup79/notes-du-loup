---
title: Alpine - Installation
summary: Création d'une VM légère Alpine.
authors: 
  - G.Leloup
date: 2024-05-19
categories: 
  - Alpine
---

## VM Alpine sous Xfce4

**Mai 2024.**

Alpine Linux est une distribution **légère** qui est essentiellement utilisée sur les systèmes embarqués, les conteneurs et les machines virtuelles.

Elle utilise **apk** comme gestionnaire de paquets et **OpenRC** comme gestionnaire de services.

Références exploitées pour la création de la VM :  
[Site Linuxtricks.fr](https://www.linuxtricks.fr/wiki/alpine-linux-guide-d-installation){ target="_blank" } et [Site Alpine.org](https://www.alpinelinux.org/){ target="_blank" }

### Téléchargement ISO

L'image ISO Alpine-Extended est téléchargeable ici :  
[https://www.alpinelinux.org/downloads/](https://www.alpinelinux.org/downloads/){ target="_blank" }

### Installation sur disque

Selon l'hyperviseur utilisé, créer une VM Alpine et démarrer celle-ci depuis l'ISO téléchargée ci-dessus.

Une demande de **login** s'affiche après quelques secondes, entrer l'utilisateur **root**.

Lancer ensuite la configuration générale :

```bash
setup-alpine
```

Le setup comprend les étapes suivantes :  
\- Configuration du clavier  
\- Création du nom d'hôte de la VM  
\- Configuration du réseau  
\- Création du MDP root  
\- Configuration du fuseau horaire  
\- Configuration éventuel d'un proxy  
\- Choix du miroir pour les paquets (entrer **f**)  
\- Création d'un nouvel utilisateur  
\- Choix du serveur SSH à utiliser  
\- Choix du disque à partitionner (entrer **sda**)  
\- Type de partitionnement (entrer **sys**)

L'installation sur le disque **sda** commence, une fois celle-ci terminée, il est demandé de rebooter.

La VM est alors prête à être utilisée, l'installation de l'OS Alpine est donc plutôt simple.

<!-- more -->

### Gestion des paquets

La liste des dépôts configurés de base se trouve dans le fichier /etc/apk/repositories.

MAJ de la liste des paquets en tant que **root** :

```bash
apk update
```

MAJ des paquets installés :

```bash
apk upgrade
```

Installation d'un paquet :

```bash
apk add nom-du-paquet
```

Désinstallation d'un paquet :

```bash
apk del nom-du-paquet
```

Rechercher si un paquet est installé :

```bash
apk list nom-du-paquet
```

Afficher la version d'un paquet installé :

```bash
apk version nom-du-paquet
```

Rechercher un paquet disponible dans les dépôts :

```bash
apk search nom-du-paquet
```

Lire les informations concernant un paquet :

```bash
apk info nom-du-paquet
```

### Bureau Xfce4 de base

Il est possible d'installer **Xfce4** qui est un environnement de bureau graphique léger.

Pour cela, commencer par éditer le fichier des dépôts :

```bash
vi /etc/apk/repositories 
```

et activer le dépôt Community en décommantant la ligne concernée.

Ensuite, mettre à jour le système :

```bash
apk update
apk upgrade
```

Installer les locales de base incluant le français :

```bash
apk add musl-locales
```

Installer le serveur graphique Xorg :

```bash
setup-xorg-base
```

Lister et installer le pilote vidéo approprié pour la carte graphique :

```bash
apk search xf86-video
apk add xf86-video-vmware
```

Ex : Le pilote **vmware** fonctionne correctement sous VirtualBox.

Lister et installer le pilote approprié pour les périphériques d'entrée :

```bash
apk search xf86-input
apk add xf86-input-libinput
```

Ex : Le pilote **libinput** gère les claviers/souris sous Xorg/Wayland.

Installer enfin le bureau graphique **Xfce4** :

```bash
apk add xfce4 xfce4-terminal
```

Installer le thème et les icônes du bureau :

```bash
apk add adw-gtk3 adwaita-icon-theme adwaita-xfce-icon-theme
```

Ajouter le verrouillage de l'écran graphique :

```bash
apk add xfce4-screensaver
```

Ajouter le montage des disques et clés USB :

```bash
apk add udisks-2 gvfs
```

Lister les plugins gvfs et installer le support de Samba :

```bash
apk search gvfs-
apk add gvfs-smb gvfs-lang
```

Installer le gestionnaire de connexion graphique LightDM :

```bash
apk add lightdm-gtk-greeter
```

Ajouter les services lightdm et dbus au boot du système :

```bash
rc-update add lightdm
rc-update add dbus
```

Paramétrer le gestionnaire de périphériques udev :

```bash
setup-devd udev
```

Installer la gestion du reboot du système depuis Xfce4 :

```bash
apk add elogind polkit-elogind
rc-update add elogind
```

Installer le son et ses utilitaires :

```bash
apk add pulseaudio pavucontrol alsa-utils
```

Puis ajouter le service alsa au boot du système :

```bash
rc-update add alsa
```

Installer pour Xfce4 le plugin Pulseaudio :

```bash
apk add xfce4-pulseaudio-plugin
```

Ajouter les locales disponibles pour les paquets installés :

```bash
apk add lang
```

Gérer la locale par défaut en créant le fichier 99-fr.sh :

```bash
vi /etc/profile.d/99-fr.sh
```

et en y insérant le contenu suivant :

```markdown
LANG=fr_FR.UTF-8
LC_CTYPE=fr_FR.UTF-8
LC_NUMERIC=fr_FR.UTF-8
LC_TIME=fr_FR.UTF-8
LC_COLLATE=fr_FR.UTF-8
LC_MONETARY=fr_FR.UTF-8
LC_MESSAGES=fr_FR.UTF-8
LC_ALL=
```

Gérer le clavier français pour l'interface graphique :

```bash
mkdir /etc/X11/xorg.conf.d
vi /etc/X11/xorg.conf.d/99-keyboard.conf
```

Insérer le contenu suivant :

```markdown
Section "InputClass"
    Identifier "keyboard"
    Option "XkbLayout" "fr"
    Option "XkbVariant" "oss"
    Option "XkbOptions" "compose:menu"
    MatchIsKeyboard "on"
EndSection
```

C'est fini, rebooter le système et vérifier le bon fonctionnement sous Xfce4.

### Serveur XRDP

Installer le serveur :

```bash
apk add xrdp xorgxrdp
```

Démarrer les services associés au serveur :

```bash
rc-service xrdp start
rc-service xrdp-sesman start
```

Pour finir, gérer le démarrage automatique du serveur :

```bash
apk rc-update add xrdp
apk rc-update add xrdp-sesman
```

### Applications ajoutées

firefox (navigateur Web)  
qutebrowser (navigateur Web léger)  
mousepad (éditeur de textes)  
nano (éditeur de textes en console)  
claws-mail (e-mail graphique)  
btop (moniteur système)

### Bilan

Positif, car moins de 2 Go de disque occupé ainsi que moins de 500 Mo de mémoire utilisée avec qutebrowser ouvert.

**Fin.**
