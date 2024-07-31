---
title: "XRDP – Astuces / Debian"
summary: Modifications diverse pour améliorer le confort d'utilisation du protocole RDP.
authors: 
  - G.Leloup
date: 2023-12-10
categories: 
  - 07 Accès RDP + VNC + SSH
---

<figure markdown>
  ![Image Pixabay - Gerd Altmann/geralt](../images/2022/01/astuce.jpg){ width="430" }
</figure>

## Distributions Debian 11 et 12

Astuces facilitant l'exploitation du service xrdp.

1. [Supprimer l'alerte "Périphérique ... couleurs"](#lien1)
2. [Modifier les connections de NetworkManager](#lien2)
3. [Autoriser le Reboot/Shutdown sous Xfce4](#lien3)

### Périphérique gestion couleurs {#lien1}

A l'ouverture des connexions sur le serveur xrdp, Debian renvoie systématiquement cette alerte :

```markdown
Il est nécessaire de s'authentifier pour créer un périphérique avec gestion de couleurs
```

Il suffit alors de s'authentifier pour pouvoir continuer.

Modifier les droits d'accès au démon colord à l'aide de l'outil polkit supprimera cette alerte.

Pour cela, créez un fichier de nom 45-allow-colord.pkla :

```bash
cd /etc/polkit-1

sudo nano localauthority/50-local.d/45-allow-colord.pkla
```

et entrez le contenu suivant :

```bash
[Allow Colord all Users]
Identity=unix-user:*
Action=org.freedesktop.color-manager.create-device;\
org.freedesktop.color-manager.create-profile;\
org.freedesktop.color-manager.delete-device;\
org.freedesktop.color-manager.delete-profile;\
org.freedesktop.color-manager.modify-device;\
org.freedesktop.color-manager.modify-profile
ResultAny=no
ResultInactive=no
ResultActive=yes
```

<!-- more -->

Le caractère \ indique d'écrire le tout sur une seule ligne.

Relancez polkit pour prendre en compte le fichier .pkla :

```bash
sudo systemctl restart polkit
```

L'alerte ne devrait plus apparaître.

### Connexions Network Manager {#lien2}

Un utilisateur du groupe sudo connecté au travers du serveur xrdp ne peut pas modifier les paramètres d'une connexion réseau configurée depuis NetworkManager.

Comme pour l'astuce précédente, il suffit de créer les droits nécessaires pour régler ce souci.

Créez le fichier 50-allow-network-manager.pkla :

```bash
cd /etc/polkit-1

sudo nano localauthority/50-local.d/50-allow-network-manager.pkla
```

et entrez le contenu suivant :

```bash
[Network Manager Utilisateurs sudo]
Identity=unix-group:sudo
Action=org.freedesktop.NetworkManager.*
ResultAny=auth_admin
ResultInactive=auth_admin
ResultActive=auth_admin
```

Ajoutez optionnellement dans NetworkManager.conf :

```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```

le contenu suivant à l'intérieur de la section [main] :

```bash
[main] 
auth-polkit=true
```

Relancez polkit pour prendre en compte le fichier .pkla :

```bash
sudo systemctl restart polkit
```

et NetworkManager pour recharger sa configuration :

```bash
sudo systemctl restart NetworkManager
```

Modifier les connexions de NetworkManager sous xrdp devrait maintenant être possible.

### Reboot/Shutdown sous Xfce4 {#lien3}

Les menus Eteindre et Déconnexion... du bureau Xfce4 sont grisés sous xrdp, donc inactifs.

Pour y remédier, créez un fichier 55-allow-reboot.pkla :

```bash
cd /etc/polkit-1

sudo nano localauthority/50-local.d/55-allow-reboot.pkla
```

et entrez le contenu suivant :

```bash
[Autoriser le shutdown depuis XRDP]
Identity=unix-user:*
Action=org.freedesktop.login1.power-off
ResultAny=yes

[Autoriser le reboot depuis XRDP]
Identity=unix-user:*
Action=org.freedesktop.login1.reboot
ResultAny=yes
```

Redémarrez le système :

```bash
sudo reboot
```

<center>---------- Fin ----------</center>
