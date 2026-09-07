---
title: Administrez votre réseau virtuel !
summary: Présentation du blog.
author: G.Leloup
date: 2026-05-17
---

<figure markdown>
  ![Synoptique - Réseau virtuel](blog/images/2026/01/reseau-virtuel.webp){ width="430" }
</figure>

## Robot de bienvenue

![Image - Lecteur des nouveautés](blog/images/2025/12/robot-bienvenue.webp){ align=left }

&nbsp;  
Apprenez comment simuler un  
réseau informatique local.  
&nbsp;  
&nbsp;

&nbsp;

La maquette proposée ci-dessous inclut **deux serveurs** Debian _(zones LAN et DMZ)_ ainsi qu'**un serveur** IPFire assurant le rôle de pare-feu pour le réseau _(zone WAN)_.

Les **postes de travail** de la zone LAN sont constitués de machines virtuelles _(VM)_ ou de conteneurs _(CTN)_ exploitant l'OS Debian..

Un **commutateur** virtuel de nom Open vSwitch et tournant sous Debian est également présent.

La construction du réseau informatique virtuel repose sur l'hyperviseur de type 2 [VirtualBox](https://www.virtualbox.org/){ target="_blank" }.

Maquette de base du réseau virtuel :

<figure markdown>
  ![Synoptique - Réseau virtuel : Flux ICMP (ping)](blog/images/2026/01/maquette-base-ipfire.png){ width="430" }
  <figcaption>Réseau virtuel : Flux ICMP (ping)</figcaption>
</figure>

[VirtualBox](https://www.virtualbox.org/){ target="_blank" } permet de créer une maquette réaliste d'un petit réseau informatique local, celle-ci s'avérant pratique pour s'initier à l'**administration réseau**.

S'il vous arrive de casser l'un des éléments du réseau, réinstallez votre **sauvegarde** créée à l'aide de la fonction "Exporter un appareil virtuel" de VirtualBox et continuez de vous amuser.

La virtualisation est un bon champ d'expérimentation pour apprendre, je consomme avec beaucoup de satisfaction et ne m'en lasse pas.

J'exploite, en particulier, les outils de virtualisation que sont VirtualBox, Proxmox et EVE-NG.

**<center>Sur ce site vous trouverez comment :</center>**

||
|:-------------:|
|Créer le réseau sous VirtualBox _(serveurs et clients)_.|
|Relier correctement les éléments entre eux.|
|Installer des services réseau tels DNS, DHCP, etc…|
|Vérifier le bon fonctionnement de l'ensemble.|

Si ce travail peut servir à d'autres qu'à moi-même, j'en serai particulièrement heureux.

**<center>Remarques générales sur la maquette :</center>**

Les VM et CTN utilisent systemd pour leur démarrage et leur gestion des services.

Le serveur IPFire dont le rôle est de sécuriser l’architecture d’un réseau informatique offre les services suivants :

- Pare-feu _(**utilisé** par le réseau virtuel)_
- Serveur Proxy
- Système de détection d’intrusion
- Serveur DHCP
- Serveur NTP _(**utilisé** par le réseau virtuel)_
- Etc …

Le commutateur Open vSwitch est installé sur une VM Debian et **non** sur le PC hôte offrant ainsi une flexibilité réseau accrue.

Prévoyez **au moins** 8 Go de RAM _(incluant la mémoire du PC hôte)_ pour une expérience fluide.

Certaines VM utilisent l’environnement de bureau graphique léger et rapide réactif **Xfce4**.

**<center>Intérêt du réseau virtuel ?</center>**

Découvrir comment créer, configurer et relier des VM et conteneurs avec VirtualBox.

Installer, configurer et tester les principaux services utilisés sur un réseau informatique.

Exploiter des outils d'administration et de supervision réseau tels Cockpit et Centreon IT-100.

Réaliser d'éventuels tests de pénétration de réseau ou de détection d'intrusion.

Etc...
