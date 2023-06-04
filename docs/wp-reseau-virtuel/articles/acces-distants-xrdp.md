---
title: "XRDP - Astuces / Debian 11"
date: "2021-12-10"
categories: 
  - "acces-distants"
---

## Astuces

Astuces facilitant l'exploitation du service xrdp.

- 1 - [Supprimer l'alerte "Périphérique ... couleurs](/acces-distants-xrdp/#xrdp-profil-couleurs)"

- 2 - [Modifier les connections de NetworkManager](/acces-distants-xrdp/#xrdp-networkmanager)

- 3 - [Autoriser le Reboot/Shutdown sous Xfce4](/acces-distants-xrdp/#xrdp-xfce4-cdes)

### 1 - Supprimer l'alerte "Périphérique ... couleurs"

A l'ouverture des connexions sur le serveur xrdp, Debian renvoie systématiquement cette alerte :

Il est nécessaire de s'authentifier pour créer un périphérique avec gestion de couleurs

Il suffit alors de s'authentifier pour pouvoir continuer.

Modifier les droits d'accès au démon colord à l'aide de l'outil polkit supprimera cette alerte.

Pour cela, créez un fichier de nom 45-allow-colord.pkla :

$ cd /etc/polkit-1
$ sudo nano localauthority/50-local.d/45-allow-colord.pkla

et entrez le contenu suivant :

\[Allow Colord all Users\]
Identity=unix-user:\*
Action=org.freedesktop.color-manager.create-device;\\
org.freedesktop.color-manager.create-profile;\\
org.freedesktop.color-manager.delete-device;\\
org.freedesktop.color-manager.delete-profile;\\
org.freedesktop.color-manager.modify-device;\\
org.freedesktop.color-manager.modify-profile
ResultAny=no
ResultInactive=no
ResultActive=yes

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Relancez polkit pour prendre en compte le fichier .pkla :

$ sudo systemctl restart polkit

L'alerte ne devrait plus apparaître.

### 2 - Modifier les connexions de Network Manager

Un utilisateur du groupe sudo connecté au travers du serveur xrdp ne peut pas modifier les paramètres d'une connexion réseau configurée depuis NetworkManager.

Comme pour l'astuce précédente, il suffit de créer les droits nécessaires pour régler ce souci.

Créez le fichier 50-allow-network-manager.pkla :

$ cd /etc/polkit-1
$ sudo nano localauthority/50-local.d/50-allow-network-manager.pkla

et entrez le contenu suivant :

\[Network Manager Utilisateurs sudo\]
Identity=unix-group:sudo
Action=org.freedesktop.NetworkManager.\*
ResultAny=auth\_admin
ResultInactive=auth\_admin
ResultActive=auth\_admin

Ajoutez optionnellement dans NetworkManager.conf :

$ sudo nano /etc/NetworkManager/NetworkManager.conf

le contenu suivant à l'intérieur de la section \[main\] :

\[main\]   
auth-polkit=true

Relancez polkit pour prendre en compte le fichier .pkla :

$ sudo systemctl restart polkit

et NetworkManager pour recharger sa configuration :

$ sudo systemctl restart NetworkManager

Modifier les connexions de NetworkManager sous xrdp devrait maintenant être possible.

### 3 - Autoriser le Reboot/Shutdown sous Xfce4

Les menus Eteindre et Déconnexion... du bureau Xfce4 sont grisés sous xrdp, donc inactifs.

Pour y remédier, créez un fichier 55-allow-reboot.pkla :

$ cd /etc/polkit-1
$ sudo nano localauthority/50-local.d/55-allow-reboot.pkla

et entrez le contenu suivant :

\[Autoriser le shutdown depuis XRDP\]
Identity=unix-user:\*
Action=org.freedesktop.login1.power-off
ResultAny=yes

\[Autoriser le reboot depuis XRDP\]
Identity=unix-user:\*
Action=org.freedesktop.login1.reboot
ResultAny=yes

Redémarrez le système :

$ sudo reboot

\---------- Fin ----------
