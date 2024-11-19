---
title: "Proxmox - Divers"
summary: Notes diverses sur Proxmox.
authors: 
  - G.Leloup
date: 2024-01-04
categories: 
  - Proxmox et NPM
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

La 2ème ligne gère les permissions du groupe _admin_.

Ajouter l'utilisateur _user-x_ au groupe _admin_ :

<!-- more -->

```bash
# pveum user modify user-x@pve -group admin
```

Le fichier modifié _user.cfg_ se siue dans /etc/pve/.

Enfin, ouvrir une session Web comme suit :  
Nom d'utilisateur : _user-x_  
Mot de passe : _mdp-user-x_  
Royaume : _Proxmox VE authentication server_

!!! note "Nota"
    La création de l'utilisateur peut également être gérée depuis l'interface Web de Proxmox.

### Interface Web en français

Ouvrir le fichier /etc/pve/datacenter.cfg et ajouter ceci :

```markdown
language: fr
```

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

### Tâche cron - Start/Stop des VM

Depuis l'interface Web, sélectionner le nom du noeud proxmox et ouvrir la console Xterm.

Lister les VM :

```bash
# qm list
```

Repérer l'ID de la VM qui sera exploitée par la tâche cron.

Editer enfin la table cron :

```bash
# crontab -e
```

et entrer le contenu suivant en fin de fichier :

```markdown
MAILTO="adresse-mail-destination"
0 23 1 * * /usr/sbin/qm start ID-VM
MAILTO="adresse-mail-destination"
0 23 3 * * /usr/sbin/qm shutdown ID-VM
```

La VM sera ainsi démarrée à 23H le premier jour de chaque mois et arrêtée à 23H le troisème jour de chaque mois.

Une notification sera envoyée simultanément à l'_adresse-mail-destination_ .

### Documentation utile

[proxmox-securisation-basique.pdf](../medias/proxmox-securisation-basique.pdf){:target="_blank"}

[proxmox8-debian12-securisation.pdf](../medias/proxmox8-debian12-securisation.pdf){:target="_blank"}

**Fin.**
