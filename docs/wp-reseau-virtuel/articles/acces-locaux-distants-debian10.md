---
title: "Contrôle à distance / Debian 10"
date: "2022-04-25"
categories: 
  - "archives"
---

## Mémento 6.1 - Accès RDP ...

Il peut être utile de pouvoir contrôler à distance les VM du réseau virtuel, notamment depuis un PC situé sur le même LAN que le PC hôte des VM. Des protocoles tels RDP, VNC ou SSH permettent cela.

Etant sur le même LAN, les connexions distantes seront validées simplement par login et MDP.

### 1 - Accès distant RDP via le serveur de VirtualBox

Par défaut, le serveur Remote Desktop Protocol fourni par VirtualBox utilise le port TCP 3389.

#### _1.1 - Configuration pour un accès sur srvlan_

Panneau gauche de VirtualBox, sélectionnez la VM :  
\> Menu Configuration  
\> Affichage > Bureau à distance  
\> Cochez Activer le serveur  
\> Port serveur > Affectez un numéro de port, Ex : 5016  
\> OK

[![Capture - VirtualBox : Réglages du bureau à distance](/wp-content/uploads/2019/05/virtualbox-bureau-distance-430x303.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/05/virtualbox-bureau-distance.png)

VirtualBox : Réglages du bureau à distance

#### _1.2 - Test de connexion_

\- Depuis un PC distant sous Windows  
Créez, sur celui-ci, les 3 routes statiques permanentes qui permettront de joindre les VM :

\[C:\\~\] route add -p 192.168.2.0 mask 255.255.255.0 192.168.x.w

\[C:\\~\] route add -p 192.168.3.0 mask 255.255.255.0 192.168.x.w

\[C:\\~\] route add -p 192.168.4.0 mask 255.255.255.0 192.168.x.w 

192.168.x.w est l'IP de la carte réseau RED d'IPFire.  
Le paramètre \-p rend la route statique permanente.

Menu Windows :  
\> Accessoires Windows ou Outils Windows  
\> Connexion Bureau à distance

Activez la connexion RDP comme montré ci-dessous :

![Capture - Bureau à Distance de Windows : Connexion IP:port sur PC hôte de VirtualBox](/wp-content/uploads/2020/03/acces-distant-rdp_1.jpg)

Bureau à distance : Connexion IP:port sur PC hôte de VirtualBox

[![Capture - Bureau à Distance de Windows : Fenêtre de login retournée par srvlan](/wp-content/uploads/2020/03/acces-distant-rdp_2-430x254.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2020/03/acces-distant-rdp_2.jpg)

Bureau à Distance : Fenêtre de login retournée par srvlan

[![Capture - Bureau à Distance de Windows : Session RDP ouverte sur srvlan](/wp-content/uploads/2020/03/acces-distant-rdp_4-430x254.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2020/03/acces-distant-rdp_4.jpg)

Bureau à Distance : Session RDP ouverte sur srvlan

\- Depuis un PC distant sous Linux  
Créez temporairement les mêmes routes statiques :

$ sudo ip route add 192.168.2.0/24 via 192.168.x.w proto static metric 100

$ sudo ip route add 192.168.3.0/24 via 192.168.x.w proto static metric 100

$ sudo ip route add 192.168.4.0/24 via 192.168.x.w proto static metric 100  

ou créez celles-ci comme routes permanentes :  
\- Si NetworkManager utilisé, c'est depuis son interface.  
\- Si fichier interfaces utilisé, ajoutez-lui ces 3 lignes :

up ip route add 192.168.2.0/24 via 192.168.x.w dev enp0s3
up ip route add 192.168.3.0/24 via 192.168.x.w dev enp0s3
up ip route add 192.168.4.0/24 via 192.168.x.w dev enp0s3

Adaptez en conséquence le nom de l'interface réseau.

Puis installez le client réseau Remmina disponible sur bien des distributions et lancez celui-ci.  
  
Entrez ensuite dans le champ de connexion rapide :  
192.168.x.y:5016  
  
192.168.x.y = IP du PC hôte des VM _([Voir la maquette](/wp-content/uploads/2018/05/maquette-base-ipfire.png))_.  
5016 est le numéro de port RDP dédié à la VM srvlan.

[![Capture - Remmina : Fenêtre de connexion RDP](/wp-content/uploads/2019/07/remmina_rdp-430x194.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/remmina_rdp.png)

Remmina : Fenêtre de connexion RDP

Appuyez sur la touche Entrée pour lancer la connexion :

[![Capture - Remmina : Session RDP ouverte sur srvlan ](/wp-content/uploads/2019/07/remmina_rdp_srvlan-430x256.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/remmina_rdp_srvlan.png)

Remmina : Session RDP ouverte sur srvlan

#### _1.3 - Remarques_

Vous devez, pour vous connecter sur les autres VM du réseau, affecter à chacune un numéro de port RDP différent, Ex : port 5017 pour la VM srvdmz.

Le serveur RDP de VirtualBox autorise des connexions directes sur les VM, pas besoin par exemple de traverser le pare-feu IPFire pour joindre srvlan.

Ceci permet notamment d'éviter un refus de connexion lié à des problèmes de configuration de table de routage ou de règle de pare-feu sur le réseau virtuel.

### 2 - Accès distant RDP sur srvlan via IPFire

Cela ne concerne pas les VM non graphiques telles IPFire et OpenvSwitch.

#### _2.1 - Installation d'un serveur RDP sur srvlan_

Installez le serveur RDP de nom xrdp :

\[srvlan@srvlan:~$\] sudo apt-get install xrdp xorgxrdp ssl-cert
\[srvlan@srvlan:~$\] sudo adduser xrdp ssl-cert

Le serveur est lancé de suite et configuré pour démarrer automatiquement au boot de la VM.

#### _2.2 - Ajout d'une règle de pare-feu pour le RDP_

L'accès distant ne fonctionnera cette fois que si vous ajoutez une règle de pare-feu dans IPFire autorisant les demandes de connexion sur le port 3389.

Accédez à l'interface graphique d'IPFire depuis le navigateur Web de srvlan :  
\> Pare-feu > Règles de pare-feu > Nouvelle règle  
  
\- Source  
\> Cochez Réseaux standards > Sélectionnez Tout  
  
\- Destination  
\> Cochez Réseaux standards > Sélectionnez Tout  
  
\- Protocole  
\> Sélectionnez TCP > Port de destination  
\> Remplissez le champ vide avec le n° de port 3389  
\> Cochez ACCEPTER  
  
\- Paramètres additionnels  
\> Remarque  
\> Entrez Connexions RDP entrantes autorisées  
  
\> Ajouter > Appliquer les changements

[![Capture - IPFire : Règle de pare-feu RDP](/wp-content/uploads/2019/07/ipfire_rdp-430x211.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/ipfire_rdp.png)

IPFire : Règle de pare-feu RDP

_Important : Fermez ensuite votre session utilisateur srvlan, le serveur xrdp refusant par défaut d'afficher le bureau d'un utilisateur ayant une session déjà ouverte._

#### _2.3 - Test de connexion_

\- Depuis un PC distant sous Windows  
Menu Windows :  
\> Accessoires Windows ou Outils Windows  
\> Connexion Bureau à distance

Entrez l'IP de srvlan 192.168.2.2 dans le champ Ordinateur et cliquez sur le bouton Connexion.

Une alerte Impossible de vérifier l'identité ... > Oui

Connexion établie, la fenêtre suivante s'affiche :

![Capture - Fenêtre de connexion du serveur xrdp](/wp-content/uploads/2019/07/ipfire_xorgrdp.png)

Fenêtre de connexion du serveur xrdp

Session > Xorg  
username > srvlan  
password : MDP de srvlan > OK

\- Depuis un PC distant sous Linux  
Utilisez de nouveau le logiciel Remmina :  
\> Entrez l'IP de srvlan 192.168.2.2  
\> Touche Entrée pour lancer la connexion

### 3 - Accès distant VNC sur srvlan via IPFire

Cela ne concerne pas les VM non graphiques telles IPFire et OpenvSwitch.

Par défaut, un serveur VNC _(Virtual Network Computing)_ écoute sur le port TCP 5900+N° Ecran.

#### _3.1 - Installation d'un serveur VNC sur srvlan_

Installez le serveur vnc4server :

\[srvlan@srvlan:~$\] sudo apt-get install vnc4server tigervnc-common 

Activez manuellement celui-ci :

\[srvlan@srvlan:~$\] vncserver  :1 -localhost no

Password > Votre MDP VNC _(Ex : vncMDPsrvlan)_  
Verify > idem  
Would you like to enter a view-only password (y/n) ? > n

Vérifiez son activation sur le port VNC 5901 :

\[srvlan@srvlan:~$\] vncserver -list 

![Capture - Serveur VNC : Port par défaut 5901 en écoute](/wp-content/uploads/2019/07/serveur_vnc_port.png)

Serveur VNC : Port par défaut 5901 en écoute

Pour info, le serveur peut être arrêté comme suit :

 \[srvlan@srvlan:~$\] vncserver -kill :1  

#### _3.2 - Lancement automatique du serveur VNC_

Pour cela, créez le service vncserver@:1.service :

\[srvlan@srvlan:~$\] cd /etc/systemd/system
\[srvlan@srvlan:~$\] sudo nano vncserver@:1.service

et entrez le contenu suivant :

\[Unit\]
Description=Service de contrôle à distance VNC
After=syslog.target network.target

\[Service\]
Type=forking
WorkingDirectory=/home/srvlan
User=srvlan
Group=srvlan

PIDFile=/home/srvlan/.vnc/%H%i.pid

ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i' > \\
     /dev/null 2>&1
ExecStart=/usr/bin/vncserver -localhost no -depth 16 \\
     -geometry 1280x720 %i
ExecStop=/usr/bin/vncserver -kill %i

\[Install\]
WantedBy=multi-user.target

Le caractère \\ indique d'écrire la Cde sur une seule ligne.

Activez le service :

\[srvlan@srvlan:~$\] sudo systemctl daemon-reload
\[srvlan@srvlan:~$\] sudo systemctl start vncserver@:1
\[srvlan@srvlan:~$\] sudo systemctl status vncserver@:1
\[srvlan@srvlan:~$\] sudo systemctl enable vncserver@:1

Pour finir, redémarrez srvlan :

\[srvlan@srvlan:~$\] sudo reboot

et vérifiez l'activation auto du serveur VNC :

\[srvlan@srvlan:~$\] vncserver -list  

#### _3.3 - Ajout d'une règle de pare-feu pour le VNC_

Accédez à l'interface graphique d'IPFire depuis le navigateur Web de srvlan :  
\> Pare-feu > Règles de pare-feu > Nouvelle règle  
  
\- Source  
\> Cochez Réseaux standards > Sélectionnez Tout  
  
\- Destination  
\> Cochez Réseaux standards > Sélectionnez Tout  
  
\- Protocole  
\> Sélectionnez TCP > Port de destination  
\> Remplissez le champ vide avec le n° de port 5901  
\> Cochez ACCEPTER  
  
\- Paramètres additionnels  
\> Remarque  
\> Entrez Connexions VNC entrantes autorisées  
  
\> Ajouter > Appliquer les changements

[![Capture - IPFire : Règle de pare-feu VNC](/wp-content/uploads/2019/07/ipfire_vnc-430x245.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/ipfire_vnc.png)

IPFire : Règle de pare-feu VNC

#### _3.4 - Test de connexion_

\- Depuis un PC distant sous Windows  
Téléchargez [ici](https://www.realvnc.com/fr/connect/download/viewer/) le client VNC Viewer de RealVNC.

Effectuez son installation et démarrez le logiciel.

Configurez ensuite une connexion pour joindre srvlan :  
\- Menu Fichier  
\> Nouvelle connexion...

Une fenêtre Propriétés s'ouvre.  
\- Onglet Général  
Paramétrez VNC Viewer comme ci-dessous :

[![Capture - VNC Viewer : Paramètres onglet Général](/wp-content/uploads/2019/07/vnc_srvlan_1-318x430.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/vnc_srvlan_1.png)

VNC Viewer : Paramètres onglet Général

\- Onglet Options  
Paramétrez VNC Viewer comme ci-dessous :

[![Capture - VNC Viewer : Paramètres onglet Options](/wp-content/uploads/2019/07/vnc_srvlan_2-318x430.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/vnc_srvlan_2.png)

VNC Viewer : Paramètres onglet Options

Résultat, VNC Viewer montre la nouvelle connexion :

[![Capture - VNC Viewer : Connexion VNC créée pour srvlan](/wp-content/uploads/2019/07/vnc_srvlan_3-430x261.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/vnc_srvlan_3.png)

VNC Viewer : Connexion VNC créée pour srvlan

Test de connexion :  
\> Double-clic sur l'icône srvlan

Une fenêtre Chiffrement s'ouvre :  
\> Cochez Ne plus afficher cet avertissement  
\> Continuer  
  
Une fenêtre Authentification s'ouvre :  
\> Entrez votre MDP VNC pour srvlan > OK

Connexion établie, la fenêtre suivante s'affiche :

[![Capture - VNC Viewer : Session VNC ouverte sur srvlan](/wp-content/uploads/2019/07/vncviewer_vnc-430x255.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/vncviewer_vnc.png)

VNC Viewer : Session VNC ouverte sur srvlan

\- Depuis un PC distant sous Linux  
a) Téléchargez [ici](https://www.realvnc.com/fr/connect/download/viewer/) le VNC Viewer de votre version Linux.  
Effectuez son installation et démarrez le logiciel.  
Procédez ensuite comme sous Windows.

b) Remmina peut également exploiter le protocole VNC.  
Cliquez sur l'icône + _(profil)_ située en haut à gauche.

Une fenêtre Profil utilisateur s'ouvre.  
Configurez la connexion VNC comme ci-dessous :

[![Capture - Remmina : Fenêtre de configuration et connexion VNC](/wp-content/uploads/2019/07/reminna_vnc-430x329.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/reminna_vnc.png)

Remmina : Fenêtre de configuration et connexion VNC

### 4 - Accès distant SSH sur srvsec (IPFire)

Par défaut, un serveur SSH _(**S**ecure **SH**ell)_ écoute sur le port TCP 22.

#### _4.1 - Activation du serveur SSH (port 222)_

Accédez à l'interface graphique d'IPFire depuis le navigateur Web de srvlan :  
\> Système > Accès SSH

Une fenêtre Accès distant s'ouvre :  
\> Cochez Accès SSH  
\> Décochez Définir le port SSH à 22 > Sauvegarder

#### _4.2 - Ajout d'une règle de pare-feu pour le SSH_

Accédez à l'interface graphique d'IPFire depuis le navigateur Web de srvlan :  
\> Pare-feu > Règles de pare-feu > Nouvelle règle  
  
\- Source  
\> Cochez Réseaux standards > Sélectionnez Tout  
  
\- Destination  
\> Cochez Réseaux standards > Sélectionnez Tout  
  
\- Protocole  
\> Sélectionnez TCP > Port de destination  
\> Remplissez le champ vide avec le n° de port 222  
\> Cochez ACCEPTER  
  
\- Paramètres additionnels  
\> Remarque  
\> Entrez Connexions SSH entrantes autorisées  
  
\> Ajouter > Appliquer les changements

[![Capture - IPFire : Règle de pare-feu SSH](/wp-content/uploads/2019/07/ipfire_ssh-430x222.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/ipfire_ssh.png)

IPFire - Règle de pare-feu SSH

#### _4.3 - Test de connexion_

\- Depuis un PC distant sous Windows  
Téléchargez le client [Putty](https://www.putty.org/) et installez celui-ci.

Démarrez et configurez l'interface de Putty comme suit :

[![Capture - Putty : Configuration SSH pour IPFire](/wp-content/uploads/2019/07/putty_config_srvsec-430x384.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/putty_config_srvsec.png)

Putty : Configuration SSH pour IPFire

Résultat de la connexion sur IPFire :

![Capture - Putty : Connexion SSH établie sur IPFire](https://familleleloup.no-ip.org/wp-content/uploads/2019/07/putty_ipfire-430x182.png)

Putty : Connexion SSH établie sur IPFire

\- Depuis un PC distant sous Linux  
a) Installez le paquet putty fourni par votre distribution.  
Procédez ensuite comme sous Windows.

b) Remmina peut également exploiter le protocole SSH.  
Cliquez sur l'icône + _(profil)_ située en haut à gauche.

Une fenêtre Profil utilisateur s'ouvre.  
Configurez la connexion SSH comme ci-dessous :

[![Capture - Remmina : Configuration SSH pour IPFire](/wp-content/uploads/2019/07/remmina_ssh-430x339.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/remmina_ssh.png)

Remmina : Configuration SSH pour IPFire

### 5 - Accès distant SSH sur ovs (OpenvSwitch)

#### _5.1 - Installation d'un serveur SSH sur ovs_

Installez le serveur OpenSSH :

\[switch@ovs:~$\] sudo apt-get install openssh-server

et contrôlez l'activation du démon SSH :

\[switch@ovs:~$\] sudo systemctl status sshd

Le résultat de la Cde doit montrer active (running).

Editez ensuite le fichier de configuration du démon :

\[switch@ovs:~$\] sudo nano /etc/ssh/sshd\_config

et remplacez la ligne #Port 22 par Port 222.

Redémarrez le serveur pour traiter la modification :

\[switch@ovs:~$\] sudo systemctl restart sshd

#### _5.2 - Test de connexion_

Effectuez une connexion à l'aide de Putty :

[![Capture - Putty : Configuration SSH pour OpenvSwitch](/wp-content/uploads/2019/07/putty_config_openvswitch-430x384.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/putty_config_openvswitch.png)

Putty : Configuration SSH pour OpenvSwitch

Résultat :

[![Capture Putty : Connexion SSH établie sur OpenvSwitch](/wp-content/uploads/2019/07/putty_openvswitch-430x168.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/07/putty_openvswitch.png)

Putty : Connexion SSH établie sur OpenvSwitch

### 6 - Bilan

Toutes les VM du réseau sont joignables à distance.

La sécurité des accès reste néanmoins perfectible :  
\- En ne travaillant pas avec des n° de port par défaut.  
\- En réglant plus finement les sources dans IPFire.  
\- En utilisant les clés publiques et privées de SSH.  
\- Etc...

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Bien, fini pour cette partie !  
Le mémento 7.1 vous attend  
pour l'installation d'un serveur  
DNS statique sur srvlan.

[Mémento 7.1](/dns-statique/)
