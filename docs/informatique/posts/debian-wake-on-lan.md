---
title: Wake On LAN
summary: Démarrage à distance au travers d'une carte réseau.
authors: 
  - G.Leloup
date: 2025-03-29
categories: 
  - Debian
  - Windows
---

## Réveil/Arrêt à distance

### Debian

Au préalable, autoriser le Wake On LAN dans le BIOS du PC.

Installer ensuite le paquet _ethtool_ :

```bash
sudo apt install ethtool
```

Puis, trouver le nom de la carte réseau cible comme suit :

```bash
sudo ip a
```

Le retour montrera _eth0_, _enp2s0_, etc...  

Si _enp2s0_, vérifier l'activation ou non du Wake On LAN :

```bash
sudo ethtool enp2s0 | grep "Wake-on"
```

Exemple de retour :

```markdown
Supports Wake-on: pumbg
Wake-on: d
```

Le retour _Wake-on: ==d==_ signifie que la carte réseau prend en charge le Wake On LAN mais que celui-ci est désactivé.

Pour l'activer, créer un fichier de service de nom _wol_ :

```bash
sudo nano /etc/systemd/system/wol.service
```

et entrer ce contenu :

<!-- more -->

```markdown
[Unit]
Description=Activation du Wake On LAN

[Service]
Type=oneshot
ExecStart=/sbin/ethtool -s enp2s0 wol g

[Install]
WantedBy=basic.target
```

Puis, démarrer le service :

```bash
sudo systemctl daemon-reload
sudo systemctl start wol
```

Vérifier le résultat comme suit :

```bash
sudo ethtool enp2s0 | grep "Wake-on"
```

Le retour doit montrer ceci :

```markdown
Supports Wake-on: pumbg
Wake-on: g
```

Le _Wake-on: ==d==_ initial est remplacé par _Wake-on: ==g==_.

Finir en activant le démarrage automatique du service :

```bash
sudo systemctl enable wol
```

### Windows 11

Au préalable, autoriser le Wake On LAN dans le BIOS du PC.

Ensuite activer le Wake On LAN comme ceci :  
a) Clic sur le bouton Démarrer de Windows  

Rechercher _Gestionnaire de périphériques_  
-> Ouvrir -> Cartes réseau  

Sélectionner la bonne carte réseau, puis :  
-> Propriétés -> Gestion de l'alimentation

Cocher les 3 autorisations concernant le Wake On LAN.

b) Clic sur le bouton Démarrer de Windows

Rechercher _Services_  
-> Ouvrir

Mettre le service _Registre à distance_ sur démarrage automatique.

c) Clic droit sur le bouton Démarrer de Windows  
-> Exécuter -> Entrer regedit -> OK

Aller ensuite dans le dossier :  
_HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows/CurrentVersion/Policies/System_

Puis, créer dans celui-ci une clé DWORD appelée _LocalAccountTokenFilterPolicy_ et lui attribuer la valeur 1.

### Outil UPSNAP

\- Exemple de Cde d'arrêt à utiliser pour Debian 12 :

```bash
sshpass -p "mot-de-passe" ssh -o "StrictHostKeyChecking=no" -p numero-de-port user@192.168.x.y "systemctl poweroff"
```

La Cde ci-dessus ne fonctionne pas sous Debian 13 qui a changé les règles de sécurité, mais une tâche cron peut-être exécutée en tant que root :

```bash
su root
crontab -e
```

Entrer le contenu suivant dans la crontab :

```markdown
# Arrêt du serveur tous les jours à 21H10
10 21 * * * /usr/bin/sudo -n /usr/bin/systemctl poweroff
```

Debian 13 s'arrêtera automatiquement à 21H10.

\- Exemple de Cde d'arrêt à utiliser pour Windows :

```bash
net rpc shutdown -I 192.168.x.y -U "user%mot-de-passe"
```

Si besoin, installer le paquet _samba-common_ qui permet l'envoi de la Cde de shutdown vers un PC Windows :

```bash
apk update
apk -e info samba-common
apk add samba-common
apk -e info samba-common
```

Retour de la dernière Cde :

```markdown
samba-common
```

### Outils PRTG et Uptime Kuma

Penser à gérer une plage horaire afin de ne pas surveiller des appareils arrêtés.

#### PRTG _(Plannings)_

Voir menu Configuration  
-> Paramètres de compte -> Onglet Plannings

#### Uptime Kuma _(Maintenance)_

Voir icône utilisateur connecté  
-> Maintenance

**Fin.**
