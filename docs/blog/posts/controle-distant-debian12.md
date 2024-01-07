---
title: "Contrôle à distance - VBox/Deb12"
summary: Utilisation des protocoles de connexion RDP, VNC, SSH.
author: G.Leloup
date: 2023-12-10
categories: 
  - Accès RDP-VNC-SSH
---

<figure markdown>
  ![Image Pixabay - La commande réseau de TheDigitalArtist](../images/2024/01/acces-distants-deb12.webp){ width="430" }
</figure>

## Mémento 6.1 - RDP, VNC et SSH

Il peut être utile de pouvoir contrôler à distance les VM et les conteneurs LXC du réseau, notamment depuis un PC situé sur le même LAN que le PC hôte de VirtualBox. Des protocoles tels RDP, VNC ou SSH permettent cela.

Etant sur le même LAN, les connexions distantes seront validées simplement par login et MDP.

### RDP via le serveur de VBox

Par défaut, le serveur Remote Desktop Protocol fourni par VirtualBox utilise le port TCP 3389.

#### _- Accès RDP sur srvlan_

Panneau gauche de VirtualBox, sélectionnez la VM :  
\- - Menu de VirtualBox > Machine > Configuration  
-> Affichage > Bureau à distance  
-> Cochez Activer le serveur  
-> Port serveur > Affectez un numéro de port, Ex : 7002  
-> OK

<figure markdown>
  ![Capture - VirtualBox : Réglages du bureau à distance](../images/2024/01/vbox-bureau-distance-deb12.webp){ width="430" }
  <figcaption>VirtualBox : Réglages du bureau à distance</figcaption>
</figure>

#### _- Test de connexion_

\- - Depuis un PC distant sous Windows  
Créez, sur celui-ci, les 3 routes statiques permanentes qui permettront de joindre les VM :

```bash
route add -p 192.168.2.0 mask 255.255.255.0 192.168.x.w

route add -p 192.168.3.0 mask 255.255.255.0 192.168.x.w

route add -p 192.168.4.0 mask 255.255.255.0 192.168.x.w 
```

192.168.x.w est l'IP de la carte réseau RED d'IPFire.  
Le paramètre -p rend la route statique permanente.

Utilisez la Cde route print pour afficher la table de routage de Windows.

Menu Windows :  
-> Accessoires Windows ou Outils Windows  
-> Connexion Bureau à distance

Activez la connexion RDP comme montré ci-dessous :

<figure markdown>
  ![Capture - Bureau à distance : IP PC hôte VM:port RDP srvlan](../images/2024/01/acces-distant-rdp-win11.webp)
  <figcaption>Bureau à distance : IP PC hôte VM:port RDP srvlan</figcaption>
</figure>

<figure markdown>
  ![Capture - Bureau à distance : Login retourné par srvlan](../images/2024/01/acces-distant-rdp-bis-win11.webp){ width="430" }
  <figcaption>Bureau à distance : Login retourné par srvlan</figcaption>
</figure>

<figure markdown>
  ![Capture - Bureau à distance : Session RDP ouverte sur srvlan](../images/2024/01/acces-distant-rdp-ter-win11.webp){ width="430" }
  <figcaption>Bureau à distance : Session RDP ouverte sur srvlan</figcaption>
</figure>

\- - Depuis un PC distant sous Linux  
Créez les mêmes routes statiques comme ceci :

```bash
sudo ip route add 192.168.2.0/24 via 192.168.x.w proto static metric 100

sudo ip route add 192.168.3.0/24 via 192.168.x.w proto static metric 100

sudo ip route add 192.168.4.0/24 via 192.168.x.w proto static metric 100  
```

Pour rendre celles-ci permanentes, vous devez tenir compte du gestionnaire de réseau actif.

Si NetworkManager est utilisé, générez les routes statiques depuis son interface graphique et relancez le service réseau :

```bash
sudo systemctl restart NetworkManager
```

Si c'est le fichier réseau /etc/network/interfaces qui est utilisé, ajoutez les lignes suivantes en fin de fichier et relancez le service réseau :

```bash
up ip route add 192.168.2.0/24 via 192.168.x.w dev enp0s3
up ip route add 192.168.3.0/24 via 192.168.x.w dev enp0s3
up ip route add 192.168.4.0/24 via 192.168.x.w dev enp0s3

sudo systemctl restart networking
```

Adaptez en conséquence le nom de l'interface réseau.

Pour finir, vérifiez la prise en compte des routes :

```bash
ip route
```

Puis installez le client réseau Remmina disponible sur bien des distributions et lancez celui-ci.

192.168.x.y = IP du PC hôte des VM _([Voir la maquette](../images/2018/05/maquette-base-ipfire.png){ target="_blank" })_.  
7002 est le numéro de port RDP dédié à la VM srvlan.

<figure markdown>
  ![Capture - Remmina : Fenêtre de connexion RDP](../images/2024/01/remmina-rdp-deb12.webp){ width="430" }
  <figcaption>Remmina : Fenêtre de connexion RDP</figcaption>
</figure>

Appuyez sur la touche Entrée pour lancer la connexion :

<figure markdown>
  ![Capture - Remmina : Session RDP ouverte sur srvlan](../images/2024/01/remmina-rdp-srvlan-deb12.webp){ width="430" }
  <figcaption>Remmina : Session RDP ouverte sur srvlan</figcaption>
</figure>

#### _- Remarques_

Vous devez, pour vous connecter sur les autres VM du réseau, affecter à chacune un numéro de port RDP différent, Ex : port 6017 pour la VM srvdmz.

Le serveur RDP de VirtualBox autorise des connexions directes sur les VM, pas besoin par exemple de traverser le pare-feu IPFire pour joindre srvlan.

Ceci permet notamment d'éviter un refus de connexion lié à des problèmes de configuration de table de routage ou de règle de pare-feu sur le réseau virtuel. Cela peut aider à dépanner.

### RDP via IPFire

Cela ne concerne pas les VM non graphiques telles IPFire, OpenvSwitch et les conteneurs LXC.

#### _- Serveur RDP sur srvlan_

Installez le serveur RDP de nom xrdp :

```bash
[srvlan@srvlan:~$] sudo apt install xrdp
[srvlan@srvlan:~$] sudo adduser xrdp ssl-cert
```

Le serveur est lancé de suite et configuré pour démarrer automatiquement au boot de la VM.

Complément pour les curieux : Suivre les astuces facilitant l'exploitation du service xrdp.

#### _- Règle de pare-feu pour le RDP_

L'accès distant ne fonctionnera cette fois que si vous ajoutez une règle de pare-feu dans IPFire autorisant les demandes de connexion sur le port 3389.

Accédez à l'interface graphique d'IPFire depuis le navigateur Web de srvlan :  
-> Pare-feu > Règles de pare-feu > Nouvelle règle

\- Zone Source  
-> Cochez Réseaux standards > Sélectionnez Tout

\- Zone Destination  
-> Cochez Réseaux standards > Sélectionnez Tout

\- Zone Protocole  
-> Sélectionnez TCP > Port de destination  
-> Remplissez le champ vide avec le n° de port 3389

-> Cochez ACCEPTER

\- Zone Paramètres additionnels  
-> Remarque  
-> Entrez Connexions RDP entrantes autorisées

-> Ajouter > Appliquer les changements