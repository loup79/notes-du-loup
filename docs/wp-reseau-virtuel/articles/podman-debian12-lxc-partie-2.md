---
title: "Podman - VBox/Deb12 2/2"
description: Podman, conteneurs rootfull et rootless.
author: G.Leloup
date: "2023-11-30"
categories: 
  - "openvswitch-conteneurs"
---

<figure markdown>
  ![Logo de Podman](../wp-content/uploads/2023/11/podman-logo.webp){ width="430" }
</figure>

## Mémento 5.2 - Conteneurs LXC

Vouss allez à présent créer un conteneur plus sécurisé dit rootless, celui-ci diminuant le risque de voir l'utilisateur root du conteneur obtenir tous les droits sur l'hôte ovs.

Le conteneur rootless _(non privilégié)_ se veut plus isolé de l'hôte LXC qu'un conteneur rootfull.

### 6 - Conteneur ctn3 en mode rootless

Pour le raccordement réseau d'un conteneur rootless, Podman utilise par défaut le paquet slirp4netns qui fournit une interface réseau virtuelle de type Tun/Tap.

Le conteneur partage alors le même espace de noms réseau que l'hôte, ce qui signifie qu'il partage la même interface réseau, les mêmes tables de routage, etc...

Le conteneur rootless ne disposant pas par défaut d'une adresse IP, c'est le mappage de ports qui est utilisé pour joindre celui-ci.

!!! Nota

    Depuis la version 4.0, il est possible de faire appel à netavark pour configurer le réseau _(non traité ici)_.

Le conteneur ctn3 sera donc raccordé par défaut ainsi :

<figure markdown>
  ![Image - Podman : Raccordement de ctn3 sur l'hôte](../wp-content/uploads/2023/11/podman-rootless.webp){ width="430" }
  <figcaption>Podman : Raccordement de ctn3 sur l'hôte</figcaption>
</figure>

#### _6.1 - Création du conteneur Podman ctn3_

Les conteneurs rootless sont créés et accessibles sans avoir besoin d'être root ou un utilisateur du groupe sudo.

Ceci est plus sécurisé car l'on ne peut pas devenir root sur l'hôte même si l'on réussit à sortir du conteneur.

Créez et lancez le conteneur ctn3 comme suit :

```bash
podman pull docker.io/louislam/uptime-kuma:1

podman volume create uptime-kuma

podman run -dit --name ctn3 -p 3001:3001 -v uptime-kuma:/app/data uptime-kuma:1
```

Les données persistantes d'Uptime Kuma seront stockées sur le volume créé et resteront disponibles même après une suppression ou une MAJ du conteneur.

Détail des options -dit et -p :  
Le d lance le conteneur en arrière plan.  
Le i autorise le mode interactif avec le conteneur.  
Le t alloue un pseudo terminal au conteneur.  
Le p mappe le port 3001 d'ovs sur le port 3001 de ctn3.

Vérifiez le téléchargement et les créations :

```bash
podman images
podman volume ls
podman ps
```

Retour de la Cde podman ps :

<figure markdown>
  ![Capture - Podman : Conteneur ctn3 créé et démarré](../wp-content/uploads/2023/11/podman-ps-ctn3-deb12.webp)
  <figcaption>Podman : Conteneur ctn3 créé et démarré</figcaption>
</figure>

L'image, le volume et le conteneur se trouvent dans :  
/home/switch/.local/share/containers/storage/*

#### _6.2 - Interaction avec le conteneur_

Si retour OK, la partie 1 est alors terminée.

![Image - Rédacteur satisfait](../wp-content/uploads/2023/07/redacteur_satisfait.jpg "Image Pixabay - Mohamed Hassan"){ align=left }

&nbsp;  
Voilà, première étape franchie !  
La partie 2 vous attend à présent  
pour la création d'un conteneur  
Podman en mode rootless.

[Partie 2](https://familleleloup.no-ip.org/virtualbox-debian11-lxc-partie-2/)
