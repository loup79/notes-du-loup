---
title: VirtualBox - Installation
summary: Installtion du logiciel VirtualBox.
authors: 
  - G.Leloup
date: 2026-05-20
categories: 
  - 01 Hyperviseur Virtualbox
---

<figure markdown>
![Image - Outil de virtuallisation VirtualBox](../images/2025/12/logo-virtualbox.webp)
</figure>

## VirtualBox - Hyperviseur type 2

### Installation

#### _- Information générale_

VirtualBox, installé sur le système d'exploitation hôte d'un ordinateur, utilisera les ressources matérielles de celui-ci pour créer des machines virtuelles _(VM)_ pouvant fonctionner simultanément sous différents systèmes d'exploitation invités.

Le système d'exploitation hôte accèdera seul au matériel physique de l'ordinateur, les systèmes d'exploitation invités des machines virtuelles utiliseront du matériel générique simulé.

Le tout fonctionnera de manière sécurisée, les systèmes d'exploitation invités n'interagiront pas directement avec le système d'exploitation hôte et n'interagiront pas entre eux.

#### _- Téléchargements_

_Le Homelab IPFire + Debian sera créé avec VBox 7.x.y._

<!-- more -->

Accédez au site [VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" } et téléchargez ces 2 fichiers :

\- L' exécutable " VirtualBox 7.x.y platform packages ", adapté à votre PC hôte.

\- L' extension " VirtualBox 7.x.y Extension Pack ", toutes plateformes.

#### _- Installation de VirtualBox 7_

Lancez l'exécutable et acceptez d'ajouter les cartes réseau qui vous seront proposées.

Démarrez ensuite VirtualBox et ajoutez le pack d'extension comme suit :  
\- - Menu Fichier  
\-> Outils  
\-> Extension -> Cliquez sur l'icône +  
\-> Sélectionnez l'extension téléchargée plus haut  
\-> Bouton Ouvrir -> Bouton Installation  
\-> Acceptez la licence

Le pack d'extension doit s'installer normalement.

Ce pack inclut les guest additons, ceux-ci permettant :  
\- d'ouvrir depuis une VM un dossier partagé par l'hôte.  
\- d’ajuster la taille des écrans des VM sur celui de l'hôte.  
\- d'effectuer des copier/coller entre les VM.  
\- d'effectuer des copier/coller entre les VM et l'hôte.  
\- d'ignorer la touche CTRL droite appelée touche Hôte.  
\- etc…

Vous pouvez à présent créer la première VM du réseau virtuel. Suivez pour cela la _liste des mémentos_ du site.

[Liste des mémentos](../../page-liste-des-mementos.md){ .md-button .md-button--primary }

### Pour aller plus loin _(VM créées)_

**a)** Lancer une VM sans ouvrir sa fenêtre graphique :

-- Hôte **Windows** --

```text
C:\...> cd \"Program Files"\Oracle\Virtualbox
C:\...>.\VBoxManage modifyvm "nom-vm" --defaultfrontend headless
```

Il faut, afin que la VM en mode headless puisse rester active même si l'utilisateur courant de Windows ferme sa session, créer une clé de registre Windows.

Pour cela, lancer la Cde regedit.exe à l'intérieur du champ du menu Exécuter de Windows, puis :  
-> HKEY_LOCAL_MACHINE\Software\Oracle\VirtualBox

Créer une clé REG_DWORD de nom VBoxSDS avec la valeur 1 et pour finir redémarrer Windows.

-- Hôte **Linux** --

```bash
VBoxManage modifyvm "nom-vm" --defaultfrontend headless
```

**b)** Lancer automatiquement une VM au boot de l'hôte :

-- Hôte **Windows** --

```text
C:\...> cd \Users\user-x\.VirtualBox
```

Créer un fichier de nom autostart.properties contenant :

```markdown
# La politique par défaut est de refuser le démarrage  
# d'une VM "deny", l'autre est de l'autoriser "allow".
default_policy = deny

# L'utilisateur user-x ci-dessous est autorisé à démarrer  
# des VM mais ceci après un délai de 30 secondes
user-x = {
allow = true
startup_delay = 30
}
```

et configurer la VM _nom-vm_ comme ceci :

```text
C:\...> cd \"Program Files"\Oracle\Virtualbox
C:\...>.\VBoxManage modifyvm "nom-vm" --autostart-enabled on --autostart-delay 180
```

Lancer ensuite la Cde sysdm.cpl à l'intérieur du champ du menu Exécuter de Windows, puis :  
-> Onglet Paramètres système avancés  
-> Bouton Variables d'environnement...  
-> Variables système > Bouton Nouvelle...

Entrer ce contenu pour la nouvelle variable Windows :  
Nom: _VBOXAUTOSTART_CONFIG_  
Valeur: C:\Users\user-x\\.VirtualBox\autostart.properties

Finir en activant le service VBoxAutostartSvc :

```txt
C:\...>.\VBoxAutostartSvc install --user=user-x
```

-- Hôte **Linux** --

Editer ou créer le fichier /etc/default/virtualbox et ajouter ceci :

```markdown
# Démarrage auto des VM
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/autostartvm.cfg
```

Créer le fichier /etc/vbox/autostartvm.cfg et entrer cela :

```markdown
default_policy=deny
user-x={allow=true startup_delay=30}
```

Ajouter l'utilisateur user-x au groupe vboxusers :

```bash
sudo usermod -aG vboxusers user-x
```

Affecter le dossier /etc/vbox/ au groupe vboxusers :

```bash
sudo chgrp vboxusers /etc/vbox
```

Autoriser le groupe vboxusers à écrire dans le dossier :

```bash
sudo chmod g+w /etc/vbox
```

Pour éviter que le répertoire ne soit modifié ou supprimé par d'autres utilisateurs, à l'exception du propriétaire ou de l'utilisateur root, définir le bit collant :

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

**c)** Activer la virtualisation imbriquée sur une VM :

Celle-ci, autrement appelée nested virtualization VT-x/AMD-V, fait référence à de la virtualisation qui s'exécute dans un environnement déjà virtualisé.

A appliquer si nécessaire :

-- Hôte **Windows** --

```text
C:\...> cd \"Program Files"\Oracle\Virtualbox
C:\...>.\VBoxManage modifyvm "nom-vm" --nested-hw-virt on
```

-- Hôte **Linux** --

```bash
VBoxManage modifyvm "nom-vm" --nested-hw-virt on
```

<center>---------- Fin ----------</center>
