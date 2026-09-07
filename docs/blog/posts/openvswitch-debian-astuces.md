---
title: "Open vSwitch - Astuces / Debian"
description: Taille console Debian et police utilisée.
authors: 
  - G.Leloup
date: 2026-03-15
categories: 
  - 06 OvS + Conteneurs LXC
---

<figure markdown>
  ![Image Pixabay - Gerd Altmann/geralt](../images/2026/01/astuce.webp){ width="430" }
</figure>

## Astuces mode console

### Taille de la console

Au boot de la VM, écran bleu d'accueil Debian affiché, appuyez sur la touche c.

Le prompt grub apparaît, entrez la Cde vbeinfo.

Le résultat montre la liste des modes vidéo disponibles.

Notez celui qui vous intéresse, Ex : 1152x864x16.

Appuyez ensuite sur la touche Escape pour quitter grub puis la touche Entrer pour lancer la VM.

Une fois lancée, éditez le fichier de configuration grub :

```bash
[switch@ovs] sudo nano /etc/default/grub
```

puis décommentez et modifiez cette ligne comme suit :

<!-- more -->

```markdown
GRUB_GFXMODE=1152x864x16
```

Enfin, ajoutez juste en dessous de celle-ci :

```markdown
GRUB_GFXPAYLOAD_LINUX=keep
```

Demandez à grub de traiter la modification et rebootez :

```bash
[switch@ovs] sudo update-grub
[switch@ovs] sudo reboot
```

Vous devriez constater le changement de taille de la console Open vSwitch.

### Police de caractère

Lancez la Cde suivante :

```bash
[switch@ovs] sudo dpkg-reconfigure -plow console-setup
```

et sélectionnez les paramètres de la police dont celui de sa taille.

Exemple :  
Codage à utiliser sur la console -> UTF-8  
Jeux de caractères à gérer -> latin1 et latin5 ...  
Police de caractères pour la console -> TerminusBold  
Taille de la police -> 12x24

Le fichier console-setup se situe dans le dossier /etc/default/.  
Les polices se situent dans le dossier /usr/share/consolefonts/.

Rebootez la VM pour traiter la modification :

```bash
[switch@ovs] sudo reboot 
```

Vous devriez constater le changement de police de caractère.

<center>---------- Fin ----------</center>
