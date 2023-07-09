---
title: "Installation"
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

### \- Information générale

VirtualBox, installé sur le système d'exploitation hôte d'un ordinateur, utilisera les ressources matérielles de celui-ci pour créer des machines virtuelles _(VM)_ pouvant fonctionner simultanément sous différents systèmes d'exploitation invités.

Le système d'exploitation hôte accèdera seul au matériel physique de l'ordinateur, les systèmes d'exploitation invités des machines virtuelles utiliseront du matériel générique simulé.

Le tout fonctionnera de manière sécurisée, les systèmes d'exploitation invités n'interagiront pas directement avec le système d'exploitation hôte et n'interagiront pas entre eux.

### \- Téléchargement de VirtualBox

_Le Homelab IPFire + Debian sera créé avec VBox 7.x.y._

Accédez au site [VirtualBox](https://www.virtualbox.org/wiki/Downloads) et téléchargez ces 2 fichiers :

\- L' exécutable VirtualBox 7.y.z platform packages, adapté au PC hôte.

\- L' extension VirtualBox 7.y.z Oracle VM VirtualBox Extension Pack, toutes plateformes.

### \- Installation de VirtualBox 7.x.y

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

![Image - Rédacteur satisfait](../wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay -
Mohamed Hassan"){ align=left }

&nbsp;  
C'est terminé ! Vous pouvez à  
présent créer la première VM  
du réseau virtuel. Suivez pour cela  
la liste des mémentos du site.

[Liste des mémentos](/liste-des-mementos/){ .md-button }
