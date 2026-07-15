---
title: Debian - Samba et Wsdd
description: Création d'un dossier partagé sur le réseau local.
authors: 
  - G.Leloup
date: 2026-07-14
categories: 
  - Debian
---

## Samba

### Installation

Installer le paquet samba :

```bash
sudo apt -y install samba
```

Répondre non pour la prise en compte de WINS.

### Configurer un partage

Editer le fichier de configuration de Samba :

```bash
cd /home/user-x
sudo nano /etc/samba/smb.conf
```

et ajouter ceci dans la section global sous la ligne workgroup = WORKGROUP :

```markdown
[global]
---
netbios name = nom d'hôte du serveur Samba
client max protocol = SMB3
client min protocol = SMB2
client signing = mandatory #sécurité imposée par Win11
client server = mandatory
```

Commenter ensuite les 3 sections suivantes :  
[homes], [printers] et [print$].

Puis, créer en fin de fichier celle-ci pour partager le dossier Téléchargements :

<!-- more -->

```markdown
[Partage-xxx]
path = /home/user-x/Téléchargements
read only = no
browseable = yes
guest ok = no
force create mode = 0660
force directory mode = 2770
;force user = nobody
;force group = nogroup
valid users = @sambashare
```

Inclure l'utilisateur user-x dans le groupe sambashare :

```bash
sudo usermod -aG sambashare user-x
```

et l'ajouter à la base d'utilisateurs Samba :

```bash
sudo pdbedit -a user-x
```

Tester enfin la bonne syntaxe du fichier smb.conf :

```bash
sudo testparm /etc/samba/smb.conf
```

et redémarrer Samba :

```bash
sudo systemctl restart smbd
sudo systemctl status smbd
```

Si status OK, autoriser son activation au boot du système :

```bash
sudo systemctl enable smbd
```

Cde utile pour trouver la localisation des fichiers Samba :

```bash
whereis samba
```

## WSD (WS-Discovery)

Autoriser la découverte par Windows 10/11 _(WS-Discovery)_  des PC Linux situés sur le réseau local.

Pour cela installer le paquet suivant :

```bash
sudo apt install wsdd2
```

Voir le [manpage](https://manpages.debian.org/trixie/wsdd2/wsdd2.8.en.html){ target = "_blank" } pour les paramètres.

Démarrer le service wsdd2 :

```bash
sudo systemctl start wsdd2
```

Vérifier l'ouverture des ports 3702 et 5355 :

```bash
ss -tulpn | grep 3702
ss -tulpn | grep 5355
```

Port 3702 = Port du service WS-Discovery  
Port 5355 = Adresse SSDP multicast

Vérifier le statut de wsdd2 :

```bash
sudo systemctl status wsdd2
```

Si status OK, autoriser son activation au boot du système :

```bash
sudo systemctl enable wsdd2
```

Voilà, Windows 10/11 devrait découvrir les partages des PC Debian automatiquement.

**Fin.**
