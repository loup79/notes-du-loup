---
title: VirtualBox - Start auto des VM
summary: Démarrage automatique des VM sous Windows et Debain.
authors: 
  - G.Leloup
date: 2020-08-06
categories: 
  - 1 - Hyperviseur Virtualbox
---

## Démarrage automatique des VM

Lancez vos VM automatiquement lors d'un redémarrage du système Windows ou Debian.

### - - Sous Windows 11 - -

VirtualBox est installé par défaut dans :

```bash
C:\Program Files\Oracle\VirtualBox\
```

et son fichier de configuration _VirtualBox.xml_ est dans :

```bash
C:\Utilisateurs\nom-du-user\.VirtualBox\
```

#### _Fichier autostart.properties_

Créez un fichier de nom _autostart.properties_ dans le dossier _\Utilisateurs\nom-du-user\\.VirtualBox\\_ et remplissez-le avec le contenu suivant :

```bash
# La politique par défaut est de refuser "deny" 
# le démarrage d'une machine virtuelle, l'autre
# option est de l'autoriser "allow".
default_policy = deny

# L'utilisateur ci-dessous est autorisé à
# démarrer les machines virtuelles mais ceci 
# après un délai de 30 secondes
nom-du-user = {
     allow = true
     startup_delay = 30
}
# Saut de ligne obligatoire après l'acollade
```

Le service qui exploitera ce fichier s'appelle *V*irtualBox *A*utostart *S*ervice _(VBoxAutostartSvc)_.

#### _Service VBoxAutostartSvc_

Créez au préalable une variable d'environnement Windows de nom _VBOXAUTOSTART_CONFIG_.

La Cde ci-dessous peut être utilisée depuis le _Terminal(administrateur)_ de Windows pour créer temporairement celle-ci :

```bash
> set VBOXAUTOSTART_CONFIG=C:\Users\nom-du-user\.VirtualBox\autostart.properties
```

<!-- more -->

Pour rendre cette variable permanente, utilisez l'application _sysdm.cpl_ qui peut être lancée depuis le champ _Exécuter_ du menu Windows.

Une fenêtre _Propriétés système_ s'ouvre :  
-> Onglet _Paramètres système avancés_  
-> Bouton _Variables d'environnement_  
-> Zone _Variables système_  
-> Bouton _Nouvelle..._  

-> Champ _Nom de la variable_, entrez :  
_VBOXAUTOSTART_CONFIG_.

-> Champ _Valeur de la variable_, entrez :  
_C:\Users\nom-du-user\\.VirtualBox\autostart.properties_.

-> Bouton _OK_

La création de la variable _VBOXAUTOSTART_CONFIG_ peut être vérifiée depuis l'application _regedit_ en accédant à la clé _Environment_ ci-dessous :

```bash
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment
```

Ensuite, activez le service de démarrage auto des VM depuis le _Terminal(administrateur)_ de Windows comme ceci :

```bash
> cd "C:\Program Files\Oracle\VirtualBox"
> .\VBoxAutostartSvc.exe install --user=nom-du-user
```

Le MDP de l'utilisateur _nom-du-user_ sera demandé.

Pour info, le service peut être désactivé ainsi :

```bash
> .\VBoxAutostartSvc.exe delete --user=nom-du-user
```

Finir en autorisant le démarrage automatique de chacune des VM _(VM arrêtée)_ :

```bash
> cd "C:\Program Files\Oracle\VirtualBox"
> .\VBoxManage.exe modifyvm "nom-de-la-vm" --autostart-enabled on --defaultfrontend headless --autostart-delay 30
```

L'option _--defaultfrontend headless_ implique un mode de démarrage _sans fenêtre d'affichage graphique_, ceci dans le cas par exemple d'une utilisation d'un accès à distance sur la VM de type RDP ou VNC.

Prévoir environ 60 secondes d'écart entre chaque démarrage de VM.

Une VM déclarée en _autostart_ voit son fichier de configuration _*.vbox_ situé dans le dossier de celle-ci contenir la ligne suivante :

```bash
<Autostart enabled="true" delay="90" autostop="Disabled"/>
```

#### _Service VBoxSVC_

VirtualBox prend en charge l'exécution du service _VBoxSVC_ dans la session 0 _(zéro)_ de Windows.

VBoxSVC fonctionne comme un service Windows normal et permet aux VM _headless_ de continuer à fonctionner même si l'utilisateur _nom-du-user_ ferme sa session Windows.

Pour le créer et l'activer, démarrez l'application _regedit_ de Windows, puis :

-> HKEY_LOCAL_MACHINE\Software\Oracle\VirtualBox

Créez une clé _REG_DWORD_ de nom _VBoxSDS_ avec la valeur _1_ et redémarrez Windows.

C'est terminé pour Windows.

### - - Sous Debian 12 - -

#### _Fichier autostartvm.cfg_

Ouvrez le fichier _/etc/default/virtualbox_ :

```bash
sudo nano /etc/default/virtualbox
```

et ajoutez le contenu suivant :

```bash
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/autostartvm.cfg
```

Créer un fichier _autostartvm.cfg_ :

```bash
sudo nano /etc/vbox/autostartvm.cfg
```

et entrez le contenu suivant :

```bash
default_policy = deny
nom-du-user = {
allow = true
startup_delay = 10
}
# Saut de ligne obligatoire après l'acollade
```

Ajoutez l'utilisateur _nom-du-user_ au groupe _vboxusers_ :

```bash
sudo usermod -aG vboxusers nom-du-user
```

Affectez le dossier _/etc/vbox/_ au groupe _vboxusers_ :

```bash
sudo chgrp vboxusers /etc/vbox
```

Autorisez le groupe à écrire dans le dossier :

```bash
sudo chmod g+w /etc/vbox
```

Redémarrez l'hôte Debian pour la prise en compte des permissions.

#### _Service vboxautostart-service_

Virtualbox fournit dans le dossier _/lib/systemd/system/_ un service appelé _vboxautostart-service_ qui se chargera de lancer les VM configurées pour un démarrage automatique.

Commencez par autoriser le démarrage auto des VM :

```bash
VBoxManage setproperty autostartdbpath /etc/vbox
```

Puis stoppez les VM et listez celles-ci :

```bash
VBoxManage list vms
```

Accédez au dossier des VM et entrez pour chacune de celles-ci la Cde suivante :

```bash
VBoxManage modifyvm "nom-de-la-vm" --autostart-enabled on --defaultfrontend headless --autostart-delay 30
```

Prévoir environ 60 secondes d'écart entre chaque démarrage de VM.

C'est terminé pour Debian.

**Fin**.
