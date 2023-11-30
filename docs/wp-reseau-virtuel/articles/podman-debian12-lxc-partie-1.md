---
title: "OVS – Podman / Debian 12 : 1/2"
author: G.Leloup
date: "2023-11-30"
categories: 
  - "openvswitch-conteneurs"
---

[![Image Pixabay - Switch informatique de Bru-nO](../wp-content/uploads/2023/10/openvswitch_3-430x323.webp#center "Cliquez pour agrandir l'image")](../wp-content/uploads/2023/10/openvswitch_3.webp)

## Mémento 5.12 - Conteneurs LXC

LXC = LinuX Container

En virtualisation, l’isolation des VM se fait au niveau matériel _(CPU/RAM/Disque ...)_ avec un accès virtuel aux ressources de l'hôte via un hyperviseur tel VirtualBox.  
  
En conteneurisation type LXC, l'isolation des conteneurs se fait au niveau de l'OS. Ceux-ci s'exécuteront de manière native en utilisant le noyau de l'hôte et en ne prenant pas plus de mémoire que tout autre exécutable.  
  
LXC va permettre d'approcher l'architecture standard d'Open vSwitch en utilisant des interfaces réseau virtuelles de type Veth pour la liaison des conteneurs avec le switch virtuel.

La **partie 1** aborde le conteneur LXC de base dit privilégié dont root sera l'utilisateur interne.

La **partie 2** aborde un conteneur plus sécurisé dit non privilégié, celui-ci devant éviter qu'une faille de sécurité donne à l'utilisateur root du conteneur la possibilité d'obtenir les privilèges complets sur l'hôte LXC.

### 1 - LXC _(Création d'un conteneur privilégié)_

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

#### _1.3 - Configuration réseau du conteneur_

La partie réseau doit être traitée avant de démarrer ctn1.

LXC configure de base une connexion virtuelle de type veth pour raccorder ctn1 au bridge lxcbr0.

Le type veth permet de créer une interface réseau sur le bridge lxcbr0 et une autre sur ctn1.

La configuration de ctn1 doit être modifiée pour raccorder celui-ci sur le bridge br0 d'OVS.

Configuration à créer :

![Image - LXC : Configuration conteneur ctn1 vers bridge br0 d'OVS](/wp-content/uploads/2021/11/lxc-ctn1-veth-ovs.jpg)

LXC : Configuration conteneur ctn1 vers bridge br0 d'OVS

Pour cela, éditez la configuration du conteneur :

\[switch@ovs:~$\] sudo nano /var/lib/lxc/ctn1/config

et modifiez la partie réseau comme suit :

\# Network configuration
lxc.net.0.type = veth
#lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
lxc.net.0.script.up = /etc/lxc/ovsup-ctn1
lxc.net.0.script.down = /etc/lxc/ovsdown-ctn1
lxc.net.0.ipv4.address = 192.168.3.6/24
lxc.net.0.ipv4.gateway = 192.168.3.15       \# IP enp0s3 VM ovs

Créez ensuite le script ovsup-ctn1 :

\[switch@ovs:~$\] sudo nano /etc/lxc/ovsup-ctn1

et entrez le contenu suivant :

#!/bin/bash

BRIDGE="br0"
ovs-vsctl --may-exist add-br $BRIDGE
ovs-vsctl --if-exists del-port $BRIDGE $5
ovs-vsctl --may-exist add-port $BRIDGE $5

Au démarrage de ctn1, le script attachera celui-ci à OVS.

Créez le script ovsdown-ctn1 :

\[switch@ovs:~$\] sudo nano /etc/lxc/ovsdown-ctn1

et entrez le contenu suivant :

#!/bin/bash

BRIDGE="br0"
ovs-vsctl --if-exists del-port $BRIDGE $5

A l'arrêt de ctn1, le script détachera celui-ci d'OVS.

Rendez les 2 scripts exécutables :

\[switch@ovs:~$\] sudo chmod +x /etc/lxc/ovs\*

La configuration réseau de ctn1 est maintenant prête.

#### _1.4 - Démarrage du conteneur_

Pour démarrer ctn1, lancez la Cde suivante :

\[switch@ovs:~$\] sudo lxc-start -n ctn1

et vérifiez ensuite le statut du conteneur :

\[switch@ovs:~$\] sudo lxc-ls -f

![Capture - LXC : Conteneur ctn1 démarré](/wp-content/uploads/2021/11/lxc-demarrage-ctn1.jpg)

LXC : Conteneur ctn1 démarré

Vérifiez aussi la création d'une interface réseau veth :

\[switch@ovs:~$\] ip address

![Capture - LXC : Interface réseau veth...@if2 créée sur la VM ovs](/wp-content/uploads/2021/11/lxc-ovs-carte-veth.jpg)

LXC : Interface réseau veth...@if2 créée sur la VM ovs

**Remarque** : Depuis Debian 11.3, l'adresse IPv4 peut ne pas apparaître du fait de l'intervention du service systemd-networkd activé par défaut dans le conteneur.

Pour régler ce problème, procédez comme suit :

\[switch@ovs:~$\] sudo lxc-attach -n ctn1

\[root@ctn1:/#\] systemctl stop systemd-networkd
\[root@ctn1:/#\] systemctl disable systemd-networkd
\[root@ctn1:/#\] exit

\[switch@ovs:~$\] sudo reboot
\[switch@ovs:~$\] sudo lxc-start -n ctn1
\[switch@ovs:~$\] sudo lxc-ls -f

L'adresse IPv4 192.168.3.6 doit cette fois apparaître.

#### _1.5 - Exécution de Cdes dans le conteneur_

Le conteneur a été créé par défaut sans MDP root.

La Cde lxc-attach permet d’exécuter des Cdes dans un conteneur, ceci en tant que root.

Utilisez celle-ci pour vous connecter sur ctn1 :

\[switch@ovs:~$\] sudo lxc-attach -n ctn1

Un prompt root@ctn1:/# doit s'afficher.

Affichez ensuite la configuration réseau de ctn1 :

\[root@ctn1:/#\] ip address

![Capture - LXC : Le conteneur montre une interface réseau eth0@if10](/wp-content/uploads/2021/11/lxc-ctn1-carte-eth0.jpg)

LXC : Le conteneur montre une interface réseau eth0@if10

Il faut, afin que ctn1 accède à Internet, lui préciser l'IP du serveur DNS à utiliser.

Pour cela, éditez le fichier DNS resolv.conf de ctn1 :

\[root@ctn1:/#\] vim /etc/resolv.conf

et ajoutez la ligne suivante :

nameserver 192.168.x.z    \# IP de votre Box Internet

Instructions pour utiliser l'éditeur de textes vim :  
Touche i _(mode insertion)_ > Pour entrez nameserv... .  
Touche Escape > Pour quitter le mode insertion.  
Touches :wq suivies de Entrée > Pour sauvegarder.

Pour tester l'accès à Internet, installez l'éditeur nano :

\[root@ctn1:/#\] apt install nano

Enfin, utilisez la Cde exit pour quitter le conteneur.

**Remarque** : Depuis Debian 11.3, le nameserver peut être par la suite écrasé du fait de l'intervention du service systemd-resolved activé par défaut dans le conteneur.

Pour régler ce problème, procédez comme suit :

\[switch@ovs:~$\] sudo lxc-attach -n ctn1

\[root@ctn1:/#\] systemctl stop systemd-resolved
\[root@ctn1:/#\] systemctl disable systemd-resolved

\[root@ctn1:/#\] rm /etc/resolv.conf
\[root@ctn1:/#\] nano /etc/resolv.conf

Entrez ce contenu dans le nouveau fichier resolv.conf :

nameserver 192.168.x.z    \# IP de votre Box Internet

Puis, quittez ctn1, rebootez ovs et redémarrez ctn1 :

\[root@ctn1:/#\] exit

\[switch@ovs:~$\] sudo reboot
\[switch@ovs:~$\] sudo lxc-start -n ctn1
\[switch@ovs:~$\] sudo lxc-ls -f

Entrez ensuite dans ctn1 et vérifiez si le nameserver du fichier resolv.conf est correct.

#### _1.6 - Démarrage automatique du conteneur_

Il est possible de démarrer automatiquement ctn1 au boot de l'hôte LXC soit la VM ovs.

Pour cela, éditez la configuration du conteneur :

\[switch@ovs:~$\] sudo nano /var/lib/lxc/ctn1/config

et ajoutez ce contenu à la fin du fichier :

\# Démarrage auto du conteneur au bout de 10s
lxc.start.auto = 1
lxc.start.delay = 10

Puis rebootez la VM ovs et contrôlez le statut de ctn1 :

\[switch@ovs:~$\] sudo lxc-ls -f

Retour :

![Capture - LXC : Démarrage automatique  du conteneur ctn1](/wp-content/uploads/2021/11/lxc-ctn1-demarrage-auto.jpg)

LXC : Démarrage automatique du conteneur ctn1

#### _1.7 - Quelques Cdes LXC utiles_

lxc-stop -n ctn1 > Stoppe ctn1  
lxc-snapshot -n ctn1 > Crée un snapshot de ctn1  
lxc-destroy -s -n ctn1 > Détruit ctn1 et ses snapshots  
lxc-info -n ctn1 > Fournit des informations sur ctn1  
lxc-copy -n ctn1 > Crée un clone de ctn1

Utilisez la Cde man lxc pour en découvrir d'autres.

#### _1.8 - Tests divers sur le réseau virtuel_

Testez depuis l'intérieur de ctn1 les pings suivants :

\[root@ctn1:/#\] ping mappy.fr               \# Internet
\[root@ctn1:/#\] ping 192.168.2.1         \# VM srvsec
\[root@ctn1:/#\] ping 192.168.4.2         \# VM srvdmz
\[root@ctn1:/#\] ping 192.168.3.4         \# VM debian11-vm2

Tous doivent recevoir une réponse positive.

Pour finir, testez un ping depuis la VM srvsec sur ctn1 :

\[root@srvsec:~#\] ping 192.168.3.6              # Conteneur ctn1

Si retour OK, la partie 1 est alors terminée.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà, première étape franchie !  
La partie 2 vous attend à présent  
pour la création d'un conteneur  
LXC dit non privilégié.

[Partie 2](https://familleleloup.no-ip.org/virtualbox-debian11-lxc-partie-2/)
