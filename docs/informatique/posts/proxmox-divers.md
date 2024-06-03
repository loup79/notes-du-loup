---
title: "Proxmox - Divers"
summary: Notes diverses sur Proxmox.
authors: 
  - G.Leloup
date: 2024-01-04
categories: 
  - Proxmox
---

## Exploitation générale

### Ajout d'un utilisateur

Se connecter en SSH sur le serveur, puis :

```bash
# pveum user add user-x@pve
# pveum passwd user-x@pve
```

Créer ensuite un groupe de nom _admin_ :

```bash
# pveum group add admin
# pveum acl modify / -group admin -role Administrator
```

et ajouter l'utilisateur _user-x_ au groupe _admin_ :

<!-- more -->

```bash
# pveum user modify user-x@pve -group admin
```

Enfin, ouvrir une session Web comme suit :  
Nom d'utilisateur : _user-x_  
Mot de passe : _mdp-user-x_  
Royaume : _Proxmox VE authentication server_

### VNC Accès externe et clavier fr

Editer le fichier de configuration de la VM concernée :

```bash
# nano /etc/pve/qemu-server/ID-vm.conf
```

et ajouter la ligne suivante en fin de fichier :

```markdown
args: -vnc 0.0.0.0:95 -k fr
```

95 correspond au numéro d'écran, choisir de préférence un nombre entre 70 et 100, Proxmox utilisant des valeurs inférieures pour les accès NoVNC.

Pour se connecter depuis un client tel TightVNC, entrer :  

```markdown
adresse-ip-hote-de-proxmox:95
```

### VNC Résolution d'écran

L'accès au BIOS _(ou UEFI)_ d'une VM se fait en appuyant plusieurs fois sur la touche Esc lors du démarrage de celle-ci.

La résolution d'écran fournie de _base_ aux clients VNC est _1280x800_.

Pour la changer en _1360x768_, se rendre dans le BIOS de la VM, puis :  
-> Device Manager > OVMF Platform Configuration  
-> _Changer la valeur de la résolution_

### Message de subscription

Pour supprimer le message, ouvrir le fichier _proxmoxlib.js_ :

```bash
# cd /usr/share/javascript
# cd proxmox-widget-toolkit
# cp proxmoxlib.js proxmoxlib.js.bak
# nano proxmoxlib.js
```

et chercher _(CTRL+W)_ la chaine _No valid subscription_.

Retour :

```markdown
Ext.Msg.show({
  title: gettext('No valid subscription'),
```

Modifier _Ext.Msg.show_ comme suit :

```markdown
void({ //Ext.Msg.show({
  title: gettext('No valid subscription'),
```

Pour finir, redémarrer le service _pveproxy_ :

```bash
# systemctl restart pveproxy.service
```

### Documentation utile

[proxmox-securisation-basique.pdf](../medias/proxmox-securisation-basique.pdf){:target="_blank"}

[proxmox8-debian12-securisation.pdf](../medias/proxmox8-debian12-securisation.pdf){:target="_blank"}

**Fin.**
