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

Vous allez à présent créer un conteneur plus sécurisé dit rootless, celui-ci diminuant le risque de voir l'utilisateur root du conteneur obtenir tous les droits sur l'hôte ovs.

Le conteneur rootless _(non privilégié)_ se veut plus isolé de l'hôte LXC qu'un conteneur rootfull.

### 6 - Conteneur ctn3 en mode rootless

Installez le paquet podman et ses dépendances :

```bash
[switch@ovs:~$] sudo apt install podman
```



<figure markdown>
  ![Image - Podman : Raccordement des conteneurs sur OVS](../wp-content/uploads/2023/11/podman-rootfull.webp){ width="430" }
  <figcaption>Podman : Raccordement des conteneurs sur OVS</figcaption>
</figure>



Si retour OK, la partie 1 est alors terminée.

![Image - Rédacteur satisfait](../wp-content/uploads/2023/07/redacteur_satisfait.jpg "Image Pixabay - Mohamed Hassan"){ align=left }

&nbsp;  
Voilà, première étape franchie !  
La partie 2 vous attend à présent  
pour la création d'un conteneur  
Podman en mode rootless.

[Partie 2](https://familleleloup.no-ip.org/virtualbox-debian11-lxc-partie-2/)
