---
title: Prérequis
summary: Besoins en matériel et connaissances.
author: G.Leloup
date: 2019-10-30
---

## Matériel et connaissances

![Image - Lecteur des nouveautés](../images/2021/12/magicien_bis.png){ align=left }

&nbsp;  
Disposer d'un PC avec un VirtualBox  
récent déjà installé et fonctionnel.  
&nbsp;  
&nbsp;  

Le PC Windows ou Linux, hôte du réseau virtuel, doit avoir environ 200 Go d'espace de libre et de préférence être équipé des éléments suivants :

| |
| :---: |
|1 Processeur assez puissant type Intel i3 ou i5|
|8 Go de RAM _(6 Go sont à dédier au réseau virtuel)_|
  
Vous pourrez ainsi, la virtualisation consommant de la ressource processeur et mémoire vive, profiter d'une exploitation confortable, ce qui n'est pas négligeable.

4 Go de RAM sont requis pour utiliser simultanément :

| |
| :---: |
|Les 3 serveurs _srvsec_, _srvlan_ et _srvdmz_|
|Le switch OpenvSwitch (sur VM _ovs_)|
|L'une des 2 VM _debian11-vm*_|
|L'un des 2 conteneurs _ctn*_|

Connaître un minimum les lignes de Cdes du système d'exploitation Linux est un plus mais pas indispensable pour y arriver. Tout est dans les mémentos du blog.

Suivez, dans l'ordre, la liste ci-dessous pour au final jouer avec un petit réseau local virtuel.

<center>-- [LISTE DES MEMENTOS](/liste-des-mementos/) --</center>
