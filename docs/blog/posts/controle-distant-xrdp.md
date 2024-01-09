---
title: "XRDP – Astuces / Debian"
summary: Modifications diverse pour améliorer le confort d'utilisation du protocole RDP.
author: G.Leloup
date: 2023-12-10
categories: 
  - Accès RDP-VNC-SSH
---

<figure markdown>
  ![Image Pixabay - Gerd Altmann/geralt](../images/2022/01/astuce.jpg){ width="430" }
</figure>

## Distributions Debian 11 et 12

Astuces facilitant l'exploitation du service xrdp.

1. Supprimer l'alerte "Périphérique ... couleurs"
2. Modifier les connections de NetworkManager
3. Autoriser le Reboot/Shutdown sous Xfce4

### Périphérique gestion couleurs

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

Le caractère \ indique d'écrire le tout sur une seule ligne.

Relancez polkit pour prendre en compte le fichier .pkla :

```bash
sudo systemctl restart polkit
```

L'alerte ne devrait plus apparaître.

### Connexions Network Manager

### Reboot/Shutdown sous Xfce4