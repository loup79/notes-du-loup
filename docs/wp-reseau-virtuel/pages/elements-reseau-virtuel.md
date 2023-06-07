---
title: "Eléments du réseau virtuel"
date: "2018-03-02"
---

## Hyperviseur - Serveurs - Clients - Switch

Le PC ou serveur, hôte de l'hyperviseur VirtualBox, peut tourner sous Windows ou Linux.

Toutes les VM créées avec VirtualBox, hormis la VM pare-feu, supportent un OS Debian Linux.  
&nbsp;  

[![Logo - VirtualBox](../wp-content/uploads/2019/02/logo-virtualbox.jpg "Logo VirtualBox"){ align=left }](https://www.virtualbox.org)

|                                  |
| -------------------------------- |
| **Système de virtualisation :**  |
| Hyperviseur de type 2 VirtualBox |
| Dernière version                 |

![Image - Serveur Linux](../wp-content/uploads/2018/03/logo-serveur-linux.png "Image Pixabay - OpenClipart-Vectors"){ align=left }

|                         |             |
| ----------------------- | ----------: |
| **Serveurs LAN/DMZ :**  |   RAM min : |
| VM _srvlan_ et _srvdmz_ | 2 x 1024 Mo |
| Bureau Xfce4 installé   |             |

[![Logo - IPFire](../wp-content/uploads/2020/11/logo-ipfire.png "Logo IPFire"){ align=left }](https://www.ipfire.org/)

|                                    |            |
| ---------------------------------- | ---------: |
| **Serveur WAN** _(Pare-feu)_ **:** |  RAM min : |
| VM _srvsec_                        | 1 x 512 Mo |
| OS LFS IPFire                      |            |

![Logo - Linux Mascotte Tux](../wp-content/uploads/2019/02/logo-linux.png "Image Pixabay - FreeCliparts"){ align=left }

|                            |             |
| -------------------------- | ----------: |
| **Clients LAN :**          |   RAM min : |
| VM _debian1x-vm1_ et _vm2_ | 2 x 1024 Mo |
| VM _ovs_                   | 1 x 1024 Mo |

[![Logo - LXC](../wp-content/uploads/2021/12/logo-lxc.png "Logo LXC"){ align=left }](https://linuxcontainers.org)

|                               |             |
| ----------------------------- | ----------: |
| **Clients LAN** _(LXC)_ **:** | RAM d'_osv_ |
| Conteneurs _ctn1_ et _ctn2_   |    partagée |
| sur la VM _ovs_               |             |

![Image - Switch informatique](../wp-content/uploads/2019/02/logo-switch.png "Image Pixabay - OpenClipart-Vectors"){ align=left }

|                       |             |
| --------------------- | ----------: |
| **Switch virtuel**    | RAM d'_osv_ |
| Logiciel Open vSwitch {.td-bleu} |    ==partagée== {.td-bleu} |
| sur la VM _ovs_  {.td-bleu}| &nbsp;  {.td-bleu}|

RAM minimum requise pour travailler convenablement :  
\-- Environ 6 Go en utilisant simultanément les 6 VM --