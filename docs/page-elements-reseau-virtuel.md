---
title: Eléments du réseau virtuel
summary: Liste des éléments constituant le réseau virtuel.
author: G.Leloup
date: 2018-03-02
---

## VBox - Serveurs - Clients - Switch

Le PC ou serveur, hôte de l'hyperviseur VirtualBox, peut tourner sous Windows ou Linux.

Toutes les VM créées avec VirtualBox, hormis la VM pare-feu, supportent un OS Debian Linux.  
&nbsp;  

[![Logo - VirtualBox](blog/images/2019/02/logo-virtualbox.jpg "Logo VirtualBox"){ align=left }](https://www.virtualbox.org){ target="_blank" }

|                                             |                            |
| --------------------------------            | -------:                   |
| **Système de virtualisation :** {.td-bleu}  |&nbsp;  {.td-bleu}          |
| Hyperviseur de type 2 VirtualBox {.td-bleu} |&nbsp;  {.td-bleu}          |
| Dernière version   {.td-bleu}               |&nbsp;  {.td-bleu}          |

![Image - Serveur Linux](blog/images/2018/03/logo-serveur-linux.png "Image Pixabay - OpenClipart-Vectors"){ align=left }

|                         |             |
| ----------------------- | ----------: |
| **Serveurs LAN/DMZ :** {.td-bleu}  |   RAM min : {.td-bleu} |
| VM _srvlan_ et _srvdmz_ {.td-bleu} | 2 x 1024 Mo {.td-bleu} |
| Bureau Xfce4 installé  {.td-bleu}  | &nbsp;  {.td-bleu}           |

[![Logo - IPFire](blog/images/2020/11/logo-ipfire.png "Logo IPFire"){ align=left }](https://www.ipfire.org/){ target="_blank" }

|                                    |            |
| ---------------------------------- | ---------: |
| **Serveur WAN** _(Pare-feu)_ **:** {.td-bleu} |  RAM min : {.td-bleu} |
| VM _srvsec_    {.td-bleu}                    | 1 x 512 Mo {.td-bleu} |
| OS LFS IPFire  {.td-bleu}                     | &nbsp;  {.td-bleu}          |

![Logo - Linux Mascotte Tux](blog/images/2019/02/logo-linux.png "Image Pixabay - FreeCliparts"){ align=left }

|                            |             |
| -------------------------- | ----------: |
| **Clients LAN :**  {.td-bleu}         |   RAM min : {.td-bleu} |
| VM _debian1x-vm1_ et _vm2_ {.td-bleu} | 2 x 1024 Mo {.td-bleu} |
| VM _ovs_   {.td-bleu}                | 1 x 1024 Mo {.td-bleu} |

[![Logo - LXC](blog/images/2021/12/logo-lxc.png "Logo LXC"){ align=left }](https://linuxcontainers.org){ target="_blank" }

|                               |             |
| ----------------------------- | ----------: |
| **Clients LAN** _(LXC)_ **:** {.td-bleu} | RAM d'_osv_ {.td-bleu}|
| Conteneurs _ctn1_ et _ctn2_  {.td-bleu}  |    partagée {.td-bleu} |
| sur la VM _ovs_  {.td-bleu}|  &nbsp; {.td-bleu} |

![Image - Switch informatique](blog/images/2019/02/logo-switch.png "Image Pixabay - OpenClipart-Vectors"){ align=left }

|                       |             |
| --------------------- | ----------: |
| **Switch virtuel :**   {.td-bleu} | RAM d'_osv_ {.td-bleu}|
| Logiciel Open vSwitch {.td-bleu} |    partagée {.td-bleu} |
| sur la VM _ovs_  {.td-bleu}| &nbsp;  {.td-bleu}|

<center>RAM minimum requise pour travailler convenablement :  
\-- Environ 6 Go en utilisant simultanément les 6 VM --</center>  
