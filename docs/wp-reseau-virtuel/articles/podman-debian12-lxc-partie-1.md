---
title: "Podman - VBox/Deb12 1/2"
author: G.Leloup
date: "2023-11-30"
categories: 
  - "openvswitch-conteneurs"
---

[![Logo de Podman](../wp-content/uploads/2023/11/podman-logo-430x350.webp#center "Cliquez pour agrandir l'image")](../wp-content/uploads/2023/11/podman-logo.webp){ loading=lazy }

## Mémento 5.2 - Conteneurs LXC

En virtualisation, l’isolation des VM se fait au niveau matériel _(CPU/RAM/Disque ...)_ avec un accès virtuel aux ressources de l'hôte via un hyperviseur tel VirtualBox.  
  
En conteneurisation, l'isolation se fait au niveau de l'OS hôte. Les conteneurs se partagent le noyau de celui-ci et ne prennent pas plus de mémoire que nécessaire.  
  
Cette technique moins exigeante en ressources favorise l'exécution d'applications légères, ces dernières incluant tout le nécessaire pour tourner de manière autonome.

Podman qui est un moteur LinuX Container permet de créer des conteneurs en tant que root _(mode rootfull)_ et en tant qu'utilisateur standard _(mode rootless)_.

Le mode rootless offre plus de sécurité car l'utilisateur standard peut créer, exécuter et gérer des conteneurs sans processus nécessitant des privilèges root.

Podman _(POD MANager)_ permet aussi de créer des pods soit des groupes de conteneurs partageant, par exemple, un même réseau IP _(non présenté ici)_.

Podman se pose en concurrent de Docker et est pilotable graphiquement depuis l'outil Cockpit.

### 1 - Installation et configuration de Podman

Installez le paquet podman et ses dépendances :

```bash
[switch@ovs:~$] sudo apt install podman
```

Contrôlez ensuite le numéro de la version installée :

```bash
[switch@ovs:~$] podman version
```

Retour :

```markdown
Client:       Podman Engine
Version:      4.3.1
API Version:  4.3.1
Go Version:   go1.19.8
Built:        Thu Jan 1 01:00:00 1970
OS/Arch:      linux/amd64
```

La Cde podman info retournera plus de détails.

Les dossiers et fichiers à suivre se trouvent dans :  
/etc/containers/*  
/home/switch/.local/share/containers/* _(rootless)_  
/var/lib/containers/* _(rootfull)_

Vérifiez maintenant l'activation du socket de Podman :

```bash
[switch@ovs:~$] sudo systemctl status podman.socket 
[switch@ovs:~$] sudo systemctl is-enabled podman.socket
```

Retours attendus : active(listening) et enabled.

Podman a besoin de cgroup pour fonctionner.

Control groups _(cgroup)_ est une fonctionnalité du noyau Linux pour limiter, compter et isoler l'utilisation des ressources _(processeur, mémoire, disque, etc...)_.

Vérifiez le montage automatique de cgroup version 2 :

```bash
[switch@ovs:~$] mount | grep cgroup
```

Retour :

```markdown
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid...)
```

Editez à présent le fichier des dépôts registries.conf :

```bash
[switch@ovs:~$] sudo nano /etc/containers/registries.conf
```

et ajoutez à la fin de celui-ci la ligne suivante :

```bash
unqualified-search-registries = [ 'docker.io']
```

Ceci permettra de contacter automatiquement le registre _(dépôt)_ de Docker, lorsque vous exécuterez la Cde podman search ou podman pull.

Pour finir, redémarrez le service socket de Podman :

```bash
[switch@ovs:~$] sudo systemctl restart podman.socket
```

### 2 - Conteneurs ctn1/ctn2 en mode rootfull

Le raccordement réseau d'un conteneur rootfull fait appel par défaut au paquet netavark qui fournit pour cela un bridge de nom podman.

Mais pour un raccordement sur le bridge br0 d'Open vSwicth, la configuration ci-dessous propose d'exploiter les espaces de noms réseau Linux plutôt que netavark .

####_2.1 - Création des espaces de noms réseau_

Dans une configuration réseau plus ou moins complexe, le mode rootfull peut s'avérer plus adapté que le mode rootless pour notamment affecter des espaces de noms réseau et des adresses IP aux conteneurs.

Un conteneur joint à un espace de noms réseau peut communiquer avec les autres espaces de noms réseau au travers d'interfaces réseau virtuelles de type Veth.

Vous raccorderez donc les conteneurs ctn1 et ctn2 au bridge br0 d'Open vSwitch comme ceci :



!!! Nota

    Podman et Docker sont globalement compatibles car fonctionnant tous les deux avec des images conformes à la norme OCI _(Open Container Initiative)_. 

blabla

#### _1.1 - Installation de LXC_

Installez les paquets lxc et lxc-templates :

\[switch@ovs:~$\] sudo apt install lxc lxc-templates

Vérifiez ensuite l'activation du routage au sein de la VM :

\[switch@ovs:~$\] sudo sysctl net.ipv4.ip\_forward

La valeur retournée doit être égale à 1.

Vérifiez aussi la bonne activation des services LXC :

\[switch@ovs:~$\] sudo systemctl status lxc 
\[switch@ovs:~$\] sudo systemctl status lxc-net

Les statuts retournés doivent être : active(exited).

LXC a besoin que cgroup soit monté pour fonctionner.  
  
Control groups _(cgroup)_ est une fonctionnalité du noyau Linux pour limiter, compter et isoler l'utilisation des ressources _(processeur, mémoire, disque, etc...)_.  
  
Vérifiez le montage automatique de cgroup version 2 :

\[switch@ovs:~$\] mount | grep cgroup

Retour :

```
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid...)
```

Pour finir, vérifiez la nouvelle configuration réseau :

\[switch@ovs:~$\] ip address

Un nouveau bridge de nom lxcbr0 a fait son apparition :

![Capture - LXC : Bridge lxcbr0 ajouté sur la VM ovs](/wp-content/uploads/2021/11/lxc-ip-address-lxcbr0.jpg)

LXC : Bridge lxcbr0 ajouté sur la VM ovs

Ce bridge fourni par défaut avec LXC sera non exploité au profit des bridges br0 et br2 d'OVS.

#### _1.2 - Création d'un conteneur de nom ctn1_

Préambule :  
Les configurations LXC seront stockées dans /etc/lxc qui contient pour l'instant un unique default.conf.  
  
Des modèles de configuration sont disponibles dans /usr/share/doc/lxc/examples/.  
  
La création de conteneurs peut être réalisée à l'aide de templates situés dans /usr/share/lxc/templates/ ou en téléchargeant une distribution spécifique sur Internet.  
  
C'est la méthode de téléchargement qui sera utilisée ici.

Créez le conteneur de nom ctn1 comme suit :

\[switch@ovs:~$\] sudo lxc-create -n ctn1 -t download -- \\
-d debian -r bullseye -a amd64

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Si Unable to fetch GPG key ..., utilisez le dépôt Ubuntu :

\[switch@ovs:~$\] sudo \\
DOWNLOAD\_KEYSERVER="keyserver.ubuntu.com" \\
lxc-create -n ctn1 -t download -- -d debian -r bullseye -a amd64

Le résultat de la Cde lxc-create doit donner ceci :

![Capture - LXC : Conteneur créé sans échec](/wp-content/uploads/2021/11/lxc-creation-ctn1.jpg)

LXC : Conteneur ctn1 créé sans échec

La distribution téléchargée a été mise en cache dans /var/cache/lxc/download/.

Vérifiez la création et le statut stoppé du conteneur :

\[switch@ovs:~$\] sudo lxc-ls -f

![Capture - LXC : Statut du conteneur ctn1](/wp-content/uploads/2021/11/lxc-statut-ctn1.jpg)

LXC : Statut du conteneur ctn1

et afficher sa configuration par défaut :

\[switch@ovs:~$\] sudo cat /var/lib/lxc/ctn1/config

![Capture - LXC : Configuration de base du conteneur ctn1](/wp-content/uploads/2021/11/lxc-config-base-ctn1.jpg)

LXC : Configuration de base du conteneur ctn1


![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà, première étape franchie !  
La partie 2 vous attend à présent  
pour la création d'un conteneur  
LXC dit non privilégié.

[Partie 2](https://familleleloup.no-ip.org/virtualbox-debian11-lxc-partie-2/)
