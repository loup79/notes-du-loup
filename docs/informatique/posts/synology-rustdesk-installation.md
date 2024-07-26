---
title: RustDesk
summary: Outil de contrôle à distance.
authors: 
  - G.Leloup
date: 2024-07-25
categories: 
  - Synology
---

Le client de contrôle à distance RustDesk fonctionnera comme celui de TeamViewer mais en se connectant cette fois sur un serveur RustDesk personnel auto-hébergé.

## RustDesk serveur

### Préalables

Docker et Portainer installés sur le NAS Synology.  
Disposer d'un nom de domaine pour l'accès à RustDesk.  

### Installation du serveur

Suivre le document [RustDesk-Serveur-sur-Synology.pdf](../medias/synology-rustdesk-serveur.pdf){:target="_blank"}.

### Ouverture de ports

Forwarder les ports TCP/UDP 21115 à 21119 depuis la box Internet vers le NAS Synology.

## RustDesk client

### Installation du client

Télécharger le client approprié sur le site de RustDesk, fichier **.deb** pour Debian et **.msi** pour Windows 11.

Installer ensuite celui-ci qui sera exécuté en tant que service.  
Ex: /usr/lib/systemd/system/rustdesk.service pour Debian.

### Cas Debian avec Xfce4

Configurer l'autologin pour l'utilsateur local _(vérifier si utile)_ :

```bash
sudo nano /etc/lightdm/lightdm.conf
```

Décommenter et paramétrer ces 2 lignes comme suit :

```markdown
autologin-user=nom-du-user-local
autologin-user-timeout=0
```

Exporter l'environnement de l'utilisateur local _(vérifier si utile)_ :

```bash
nano .bashrc
```

Ajouter les lignes suivantes en fin de fichier :

```markdown
# Ajout pour utiliser le client Rustdesk
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$UID/bus"
export XDG_RUNTIME_DIR="/run/user/$UID"
export XDG_SESSION_TYPE=x11
```

### Cas Debian avec xrdp

Autoriser une connexion simultanée locale et distante :

```bash
sudo nano /etc/xrdp/startwm.sh
```

Ajouter les 2 lignes suivantes juste avant test -x --- :

```markdown
# Pour connexion simultanée xrdp et normale
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
# Fin
```

### Cas Debian Xfce4 sans écran

Prévoir la mise en place d'un connecteur HDMI simulant la présence d'un moniteur afin de garantir le lancement de la session locale Xfce4.

Voir dans l'onglet Général du client RustDesk si le mode "allow linux headless" est utile.

### Cde pour lire l'environnement

```bash
systemctl --user show environment
```

### Cdes pour gérer les sessions

```bash
loginctl list-sessions
loginctl show-session ID
loginctl show-session -p State ID
loginctl session-status ID

loginctl terminate-session ID
loginctl activate ID

loginctl list-users
loginctl show-user ID
```

L'ID correspond au n° de session soit 1 ou 3 ou c1, etc...

**Fin.**
