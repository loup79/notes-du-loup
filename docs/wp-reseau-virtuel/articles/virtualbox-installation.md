---
title: "Installation"
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

