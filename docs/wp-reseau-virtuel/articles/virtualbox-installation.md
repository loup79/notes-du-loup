---
title: "Installation"
date: "2020-08-08"
categories: 
  - "hyperviseur-virtualbox"
---

## VirtualBox est un hyperviseur de type 2

<table><tbody><tr><td><a href="https://www.virtualbox.org/" target="_blank" rel="noreferrer noopener"><img class="wp-image-35" style="width: 120px" src="../../wp-content/uploads/2019/02/logo-virtualbox.jpg" alt="Logo - VirtualBox" title="https://www.virtualbox.org/"></a></td><td>VirtualBox permet de créer des Machines Virtuelles <em>(VM)</em>.</td></tr></tbody></table>

### \- Information générale :

VirtualBox, installé sur le système d'exploitation hôte d'un ordinateur, utilisera les ressources matérielles de celui-ci pour créer des machines virtuelles _(VM)_ pouvant fonctionner simultanément sous différents systèmes d'exploitation invités.

Le système d'exploitation hôte accèdera seul au matériel physique de l'ordinateur, les systèmes d'exploitation invités des machines virtuelles utiliseront du matériel générique simulé.

Le tout fonctionnera de manière sécurisée, les systèmes d'exploitation invités n'interagiront pas directement avec le système d'exploitation hôte et n'interagiront pas entre eux.

### \- Téléchargement de VirtualBox :

Accédez au site [VirtualBox](https://www.virtualbox.org/wiki/Downloads) et téléchargez ces 2 fichiers :

\- L' exécutable VirtualBox x.y.z platform packages, adapté au PC hôte.

\- L' extension VirtualBox x.y.z Oracle VM VirtualBox Extension Pack, toutes plateformes.

### \- Installation de VirtualBox :

Lancez l'exécutable et ajoutez les cartes réseau qui vous seront proposées.

Démarrez ensuite VirtualBox et ajoutez le pack d'extension comme suit :  
\- Menu Fichier  
\> Paramètres...  
\> Extensions > L'icône + "Ajoute une nouvelle extension."  
\> Sélectionnez l'extension téléchargée plus haut  
\> Ouvrir > Installation > OK

Ce pack inclut les guest additons, outils permettant :  
\- d'ouvrir depuis une VM un dossier partagé par l'hôte.  
\- d’ajuster la taille des écrans des VM sur celui de l'hôte.  
\- d'effectuer des copier/coller entre les VM.  
\- d'effectuer des copier/coller entre les VM et l'hôte.  
\- d'ignorer la touche CTRL droite appelée touche Hôte.  
\- etc…

![Image - Rédacteur satisfait](../../wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan"){ align=left }

  
C'est terminé ! Vous pouvez à  
présent créer la première VM  
du réseau virtuel. Suivez pour cela  
la liste des mémentos du site.

[Liste des mémentos](/liste-des-mementos/)
