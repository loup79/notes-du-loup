---
title: "Prérequis"
date: "2019-10-30"
---

## Configuration matérielle et logicielle

![Image - Lecteur des nouveautés](https://familleleloup.no-ip.org/wp-content/uploads/2021/12/magicien_bis.png)

Disposer d'un PC avec un VirtualBox  
récent déjà installé et fonctionnel.

Le PC, hôte du réseau virtuel, doit avoir environ 200 Go d'espace de libre et de préférence être équipé des éléments suivants :

<table class="has-base-2-background-color has-background"><tbody><tr><td>1 Processeur assez puissant type Intel i3 ou i5</td></tr><tr><td>8 Go de RAM <em>(6 Go sont à dédier au réseau virtuel)</em></td></tr></tbody></table>

  
Vous pourrez ainsi, la virtualisation consommant de la ressource processeur et mémoire vive, profiter d'une exploitation confortable, ce qui n'est pas négligeable.

4 Go de RAM sont requis pour utiliser simultanément :

<table class="has-base-2-background-color has-background"><tbody><tr><td>Les 3 serveurs <em>srvsec</em>, <em>srvlan</em> et <em>srvdmz</em></td></tr><tr><td>Le switch OpenvSwitch (sur VM <em>ovs</em>)</td></tr><tr><td>L'une des 2 VM <em>debian11-vm*</em></td></tr><tr><td>L'un des 2 conteneurs <em>ctn*</em></td></tr></tbody></table>

  
Connaître un minimum les lignes de Cdes du système d'exploitation Linux est un plus mais pas indispensable pour y arriver. Tout est dans les mémentos du blog.

Suivez, dans l'ordre, la liste ci-dessous pour au final jouer avec un petit réseau local virtuel.

\-- [LISTE DES MEMENTOS](/liste-des-mementos/) --
