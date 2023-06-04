---
title: "OVS - Astuces / Debian"
date: "2019-02-20"
categories: 
  - "openvswitch-lxc"
---

## Astuces

### 1 - Taille de l'écran en mode console

Au boot de la VM, écran bleu d'accueil Debian affiché, appuyez sur la touche c.

Le prompt grub apparaît, entrez la Cde vbeinfo.

Le résultat montre la liste des modes vidéo disponibles.

Notez celui qui vous intéresse, Ex : 1152x864x16.

Appuyez ensuite sur la touche Escape pour quitter grub puis la touche Entrer pour lancer la VM.

Une fois lancée, éditez le fichier de configuration grub :

\[switch@ovs:~$\] sudo nano /etc/default/grub

puis décommentez et modifiez cette ligne comme suit :  
GRUB\_GFXMODE=1152x864x16

Enfin, ajoutez juste en dessous de celle-ci :  
GRUB\_GFXPAYLOAD\_LINUX=keep

Demandez à grub de traiter la modification et rebootez :

\[switch@ovs:~$\] sudo update-grub  
\[switch@ovs:~$\] sudo reboot

Vous devriez constater le changement de taille de la console Open vSwitch.

### 2 - Police de caractère en mode console

Lancez la Cde suivante :

\[switch@ovs:~$\] sudo dpkg-reconfigure -plow console-setup

et sélectionnez les paramètres de votre police dont celui de sa taille.

Le fichier console-setup se situe dans /etc/default/.  
Les polices se situent dans /usr/share/consolefonts/.

Rebootez la VM pour traiter la modification :

\[switch@ovs:~$\] sudo reboot 

Vous devriez constater le changement de police de caractère.

\---------- Fin ----------
