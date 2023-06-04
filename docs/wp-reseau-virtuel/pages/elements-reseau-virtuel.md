---
title: "Eléments du réseau virtuel"
date: "2018-03-02"
---

## Hyperviseur - Serveurs - Clients - Switch

Le PC ou serveur, hôte de l'hyperviseur VirtualBox, peut tourner sous Windows ou Linux.

Toutes les VM créées avec VirtualBox, hormis la VM pare-feu, supportent un OS Debian Linux.

[![Logo - VirtualBox](/wp-content/uploads/2019/02/logo-virtualbox.jpg "Logo VirtualBox")](https://www.virtualbox.org)

<table class="has-background" style="background-color:#e7f5fe"><tbody><tr><td class="has-text-align-left" data-align="left"><strong>Système de virtualisation :</strong></td></tr><tr><td class="has-text-align-left" data-align="left">Hyperviseur de type 2 VirtualBox</td></tr><tr><td class="has-text-align-left" data-align="left">Dernière version</td></tr></tbody></table>

![Image - Serveur Linux](/wp-content/uploads/2018/03/logo-serveur-linux.png "Image Pixabay - OpenClipart-Vectors")

<table class="has-background" style="background-color:#e7f5fe"><tbody><tr><td class="has-text-align-left" data-align="left"><strong>Serveurs LAN/DMZ :</strong></td><td class="has-text-align-right" data-align="right">RAM min :</td></tr><tr><td class="has-text-align-left" data-align="left">VM <em>srvlan </em>et<em> srvdmz</em></td><td class="has-text-align-right" data-align="right">2 x 1024 Mo</td></tr><tr><td class="has-text-align-left" data-align="left">Bureau Xfce4 installé</td><td class="has-text-align-right" data-align="right"></td></tr></tbody></table>

[![Logo - IPFire](/wp-content/uploads/2020/11/logo-ipfire.png "Logo IPFire")](https://www.ipfire.org/)

<table class="has-background" style="background-color:#e7f5fe"><tbody><tr><td class="has-text-align-left" data-align="left"><strong>Serveur WAN </strong><em>(Pare-feu)</em><strong> :</strong></td><td class="has-text-align-right" data-align="right">RAM min :</td></tr><tr><td class="has-text-align-left" data-align="left">VM <em>srvsec</em></td><td class="has-text-align-right" data-align="right">1 x 512 Mo</td></tr><tr><td class="has-text-align-left" data-align="left">OS LFS IPFire</td><td class="has-text-align-right" data-align="right"></td></tr></tbody></table>

![Logo - Linux Mascotte Tux](/wp-content/uploads/2019/02/logo-linux.png "Image Pixabay - FreeCliparts")

<table class="has-background" style="background-color:#e7f5fe"><tbody><tr><td class="has-text-align-left" data-align="left"><strong>Clients LAN :</strong></td><td class="has-text-align-right" data-align="right">RAM min :</td></tr><tr><td class="has-text-align-left" data-align="left">VM <em>debian1x-vm1 </em>et<em> vm2</em></td><td class="has-text-align-right" data-align="right">2 x 1024 Mo</td></tr><tr><td class="has-text-align-left" data-align="left">VM <em>ovs</em></td><td class="has-text-align-right" data-align="right">1 x 1024 Mo</td></tr></tbody></table>

[![Logo - LXC](/wp-content/uploads/2021/12/logo-lxc.png "Logo LXC")](https://linuxcontainers.org)

<table class="has-background" style="background-color:#e7f5fe"><tbody><tr><td class="has-text-align-left" data-align="left"><strong>Clients LAN </strong><em>(LXC)</em><strong> :</strong></td><td class="has-text-align-right" data-align="right">RAM d'<em>ovs</em></td></tr><tr><td class="has-text-align-left" data-align="left">Conteneurs <em>ctn1</em> et <em>ctn2</em></td><td class="has-text-align-right" data-align="right">partagée&nbsp;&nbsp;&nbsp;</td></tr><tr><td class="has-text-align-left" data-align="left">sur la VM <em>ovs</em></td><td class="has-text-align-right" data-align="right"></td></tr></tbody></table>

![Image - Switch informatique](/wp-content/uploads/2019/02/logo-switch.png "Image Pixabay - OpenClipart-Vectors")

<table class="has-background" style="background-color:#e7f5fe"><tbody><tr><td class="has-text-align-left" data-align="left"><strong>Switch virtuel :</strong></td><td class="has-text-align-right" data-align="right">RAM d'<em>ovs</em></td></tr><tr><td class="has-text-align-left" data-align="left">Logiciel Open vSwitch</td><td class="has-text-align-right" data-align="right">partagée&nbsp;&nbsp;&nbsp;</td></tr><tr><td class="has-text-align-left" data-align="left">sur la VM <em>ovs</em></td><td class="has-text-align-right" data-align="right"></td></tr></tbody></table>

RAM minimum requise pour travailler convenablement :  
\-- Environ 6 Go en utilisant simultanément les 6 VM --
