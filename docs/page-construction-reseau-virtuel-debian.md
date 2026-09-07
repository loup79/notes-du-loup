---
title: "Construction du réseau"
summary: Détail des étapes de construction du réseau.
author: G.Leloup
date: 2026-05-18
---

## Machines viruelles et services réseau

![Image - Evolution de l'homme](blog/images/2026/01/image-evolution.png){ align=left }

&nbsp;  
&nbsp;  
4 ETAPES LOGIQUES  
&nbsp;  
&nbsp;

&nbsp;

La construction suit les 4 étapes titrées ci-dessous. Les 3 premières concernent la création des différentes machines virtuelles et la quatrième concerne l'installation des services sur le réseau.

### Installation de l'outil de virtualisation

Commencez par installer le logiciel VirtualBox.

&#9755; Mémento [VirtualBox](blog/posts/virtualbox-installation.md)

### Création des serveurs virtuels

Puis créez, comme le montre la [maquette](blog/images/2026/01/maquette-base-ipfire.png), les serveurs des zones LAN, WAN et DMZ.

&#9755; Mémentos [srvlan](blog/posts/serveur-debian-srvlan-creation.md) / [srvsec](blog/posts/serveur-ipfire-srvsec-creation.md) / [srvdmz](blog/posts/serveur-debian-srvdmz-creation.md)

### Création des clients virtuels de la zone LAN

L'ensemble comprend 2 VM Debian et 1 VM Open vSwitch incluant 2 conteneurs.

&#9755; Mémentos [Clients](blog/posts/clients-debian-vm1-vm2-creation.md) / [Open vSwitch](blog/posts/openvswitch-debian-ovs-creation.md/)

### Installation des services DNS, DHCP, WEB ...

Suivez, dans l'ordre, la liste des mémentos du blog pour au final pouvoir exploiter un petit réseau local virtuel.

&#9755; [Liste des mémentos](page-liste-des-mementos.md)

<center> . . . . . . </center>

L'idéal pour réaliser tout ceci est de disposer d'un petit serveur basse consommation avec suffisamment de mémoire pour supporter la construction du réseau.

Ceci vous évitera de relancer vos VM chaque fois que vous souhaiterez travailler sur le réseau.

Le mémento [Contrôle à distance](blog/posts/controle-distant-debian.md) est là pour que vous puissiez piloter vos VM depuis un PC situé sur le même réseau local que le serveur ou PC hôte de VirtualBox.

Ce réseau virtuel est là pour aider tout débutant souhaitant disposer d'un outil lui permettant d'aborder l'administration réseau.

Bon courage !
