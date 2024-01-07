---
title: "Contrôle à distance - VBox/Deb12"
summary: Utilisation des protocoles de connexion RDP, VNC, SSH.
author: G.Leloup
date: 2023-12-10
categories: 
  - Accès RDP-VNC-SSH
---

<figure markdown>
  ![Logo de Podman](../images/2024/01/acces-distants-deb12.webp){ width="430" }
</figure>

## Mémento 6.1 - RDP, VNC et SSH

Il peut être utile de pouvoir contrôler à distance les VM et les conteneurs LXC du réseau, notamment depuis un PC situé sur le même LAN que le PC hôte de VirtualBox. Des protocoles tels RDP, VNC ou SSH permettent cela.

Etant sur le même LAN, les connexions distantes seront validées simplement par login et MDP.

### RDP via le serveur de VBox

Par défaut, le serveur Remote Desktop Protocol fourni par VirtualBox utilise le port TCP 3389.

#### _- Accès RDP sur srvlan_

Panneau gauche de VirtualBox, sélectionnez la VM :  
- - Menu de VirtualBox > Machine > Configuration...  
-> Affichage > Bureau à distance  
-> Cochez Activer le serveur  
-> Port serveur > Affectez un numéro de port, Ex : 7002  
-> OK

