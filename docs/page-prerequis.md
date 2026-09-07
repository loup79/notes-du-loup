---
title: Prérequis
summary: Besoins en matériel et connaissances.
author: G.Leloup
date: 2026-05-17
---

## Matériels et connaissances

![Image - Lecteur des nouveautés](blog/images/2025/12/robot-bienvenue.webp){ align=left }

&nbsp;  
Vous devez disposer d'un PC  
supportant un OS Windows ou Linux.  
&nbsp;  
&nbsp;

&nbsp;

Ce PC qui sera l'**hôte** du réseau virtuel doit avoir environ 100 Go d'espace disque de libre et de préférence être équipé des éléments suivants :

||
|:---:|
|1 Processeur assez puissant type Intel i5 ou Ryzen 5|
|8 Go de RAM ou + _(6 Go dédié au réseau virtuel)_|

&nbsp;
  
Vous pourrez ainsi, la virtualisation consommant de la ressource processeur et mémoire vive, profiter d'une exploitation confortable, ce qui n'est pas négligeable.

Les 6 Go de RAM sont requis pour utiliser simultanément :

||
|:---:|
|Les 3 serveurs _srvsec_, _srvlan_ et _srvdmz_|
|Le switch OpenvSwitch (sur VM _ovs_)|
|L'une des 2 VM _debian..-vm*_|
|L'un des 2 conteneurs _ctn*_|

Une configuration de 16 Go de RAM est parfaite.

&nbsp;

Connaître les commandes du système d'exploitation Linux est un plus mais pas indispensable pour arriver à construire le réseau. Tout est détaillé dans les mémentos du site.

Suivez, **dans l'ordre**, la liste ci-dessous pour au final jouer avec votre propre réseau local virtuel.

||
|:---:|
|[**LISTE DES MEMENTOS**](page-liste-des-mementos.md)|
