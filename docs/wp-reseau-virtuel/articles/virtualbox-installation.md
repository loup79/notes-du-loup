---
title: "VirtualBox - Installation"
author: G.Leloup
date: "2020-08-08"
categories: 
  - "hyperviseur-virtualbox"
---

![Image](../../wp-reseau-virtuel/wp-content/uploads/2022/08/virtualbox_os.jpg)

## VirtualBox est un hyperviseur de type 2

![Logo - VirtualBox](../wp-content/uploads/2019/02/logo-virtualbox.jpg){ align=left }

&nbsp;  
VirtualBox permet  
de créer des Machines  
Virtuelles (VM).  
&nbsp;  

### 1 - Installation

#### _1.1 - Information générale_

VirtualBox, installé sur le système d'exploitation hôte d'un ordinateur, utilisera les ressources matérielles de celui-ci pour créer des machines virtuelles _(VM)_ pouvant fonctionner simultanément sous différents systèmes d'exploitation invités.

Le système d'exploitation hôte accèdera seul au matériel physique de l'ordinateur, les systèmes d'exploitation invités des machines virtuelles utiliseront du matériel générique simulé.

Le tout fonctionnera de manière sécurisée, les systèmes d'exploitation invités n'interagiront pas directement avec le système d'exploitation hôte et n'interagiront pas entre eux.

#### _1.2 - Téléchargement de VirtualBox_

_Le Homelab IPFire + Debian sera créé avec VBox 7.x.y._

Accédez au site [VirtualBox](https://www.virtualbox.org/wiki/Downloads) et téléchargez ces 2 fichiers :

\- L' exécutable VirtualBox 7.x.y platform packages, adapté au PC hôte.

\- L' extension VirtualBox 7.x.y Oracle VM VirtualBox Extension Pack, toutes plateformes.

#### _1.3 - Installation de VirtualBox 7.x.y_

Lancez l'exécutable et acceptez d'ajouter les cartes réseau qui vous seront proposées.

Démarrez ensuite VirtualBox et ajoutez le pack d'extension comme suit :  
\- Menu Fichier  
\> Outils  
\> Extension Pack Manager > Cliquez sur l'icône +  
\> Sélectionnez l'extension téléchargée plus haut  
\> Bouton Ouvrir > Bouton Installation  
\> Acceptez la licence

Le pack d'extension doit s'installer normalement.

Ce pack inclut les guest additons, ceux-ci permettant :  
\- d'ouvrir depuis une VM un dossier partagé par l'hôte.  
\- d’ajuster la taille des écrans des VM sur celui de l'hôte.  
\- d'effectuer des copier/coller entre les VM.  
\- d'effectuer des copier/coller entre les VM et l'hôte.  
\- d'ignorer la touche CTRL droite appelée touche Hôte.  
\- etc…  
&nbsp;  

[Liste des mémentos](/liste-des-mementos/){ .md-button }

### 2 - Pour aller plus loin une fois les VM créées

**a)** Lancer une VM sans ouvrir sa fenêtre graphique :

-- Hôte **Windows** --

```bash
C:\...> cd \"Program Files"\Oracle\Virtualbox
C:\...>.\VBoxManage modifyvm "nom-vm" --defaultfrontend headless
```

Il faut, afin que la VM en mode _headless_ puisse rester active même si l'utilisateur courant de Windows ferme sa session, créer une _clé de registre_ Windows.

Pour cela, lancer la Cde _regedit.exe_ à l'intérieur du champ du menu _Exécuter_ de Windows, puis :  
-> HKEY_LOCAL_MACHINE\Software\Oracle\VirtualBox

Créer une clé REG_DWORD de nom VBoxSDS avec la valeur 1 et pour finir redémarrer Windows.
Créer une clé REG_DWORD de nom _VBoxSDS_ avec la valeur _1_ et pour finir redémarrer Windows.

-- Hôte **Linux** --

```bash
VBoxManage modifyvm "nom-vm" --defaultfrontend headless
```

**b)** Lancer _automatiquement_ une VM au boot de l'hôte :

-- Hôte **Windows** --

```bash
C:\...> cd \Users\user-x\.VirtualBox
```

Créer un fichier de nom _autostart.properties_ contenant :

```bash
# La politique par défaut est de refuser "deny" le démarrage  
# d'une VM, l'autre option est de l'autoriser "allow".
default_policy = deny

# L'utilisateur user-x ci-dessous est autorisé à démarrer  
# des VM mais ceci après un délai de 30 secondes
user-x = {
allow = true
startup_delay = 30
}
```

et configurer la VM _nom-vm_ comme ceci :

```bash
C:\...> cd \"Program Files"\Oracle\Virtualbox
C:\...>.\VBoxManage modifyvm "nom-vm" --autostart-enabled on --autostart-delay 180
```

Lancer ensuite la Cde _sysdm.cpl_ à l'intérieur du champ du menu _Exécuter_ de Windows, puis :  
-> Onglet _Paramètres système avancés_  
-> Bouton _Variables d'environnement..._  
-> Variables système > Bouton _Nouvelle..._

Entrer ce contenu pour la nouvelle variable Windows :  
Nom: VBOXAUTOSTART_CONFIG  
Valeur: _C:\Users\user-x\.VirtualBox\autostart.properties_

Finir en activant le service _VBoxAutostartSvc_ :

```bash
C:\...>.\VBoxAutostartSvc install --user=user-x
```

-- Hôte **Linux** --

Editer le fichier _/etc/default/virtualbox_ et ajouter ceci :

```bash
# Démarrage auto des VM
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/autostartvm.cfg
```

Créer le fichier _/etc/vbox/autostartvm.cfg_ et entrer cela :

```bash
default_policy = deny
user-x = {
allow = true
startup_delay = 30
}
# Saut de ligne après la parenthèse
```

Ajouter l'utilisateur _user-x_ au groupe _vboxusers_ :

```bash
sudo usermod -aG vboxusers user-x
```

Affecter le dossier _/etc/vbox/_ au groupe _vboxusers_ :

```bash
sudo chgrp vboxusers /etc/vbox
```

Autoriser le groupe _vboxusers_ à écrire dans le dossier :

```bash
sudo chmod g+w /etc/vbox
```

Pour éviter que le répertoire ne soit modifié ou supprimé par d'autres utilisateurs, à l'exception du propriétaire ou de l'utilisateur _root_, définir le _bit collant_ :

```bash
sudo chmod +t /etc/vbox
```

Redémarrer l'hôte pour le traitement des permissions.

Laisser les VM à l'arrêt et autoriser le démarrage automatique de celles-ci :

```bash
VBoxManage setproperty autostartdbpath /etc/vbox
```

Obtenir la liste des VM :

```bash
VBoxManage list vms
```

Se rendre dans le dossier des VM et entrer :

```bash
VBoxManage modifyvm "nom-vm" --autostart-enabled on --autostart-delay 180
```

**c)** Activer la _virtualisation imbriquée_ sur une VM :

Celle-ci, autrement appelée _nested virtualization VT-x/AMD-V_, fait référence à de la virtualisation qui s'exécute dans un environnement déjà virtualisé.

A appliquer si nécessaire :

-- Hôte **Windows** --

```bash
C:\...> cd \"Program Files"\Oracle\Virtualbox
C:\...>.\VBoxManage modifyvm "nom-vm" --nested-hw-virt on
```

-- Hôte **Linux** --

```bash
VBoxManage modifyvm "nom-vm" --nested-hw-virt on
```

---------- Fin ----------
