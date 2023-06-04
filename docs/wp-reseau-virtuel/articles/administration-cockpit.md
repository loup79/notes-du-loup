---
title: "Cockpit / Debian 11"
date: "2022-12-07"
categories: 
  - "outil-cockpit"
---

## Mémento 12.1 - Cockpit

Cockpit permettra au travers d'une interface Web de gérer à distance les systèmes Linux des VM du réseau virtuel, ceci en utilisant le nom de domaine loupvirtuel.fr.

La gestion des VM sera centralisée depuis un serveur Cockpit primaire installé sur srvdmz.

### 1 - Accès distant sur le réseau virtuel

Le domaine loupvirtuel.fr n'étant pas public, vous ne pouvez pas accéder au site Web du réseau virtuel depuis Internet en tapant son URL dans le champ adresse d'un navigateur Web.

En revanche, vous pouvez simuler cet accès Internet depuis un PC situé sur le réseau local de votre domicile.

Par exemple, pour un PC Windows :

\- Etape 1  
Entrez sur ce PC, comme dans le mémento [Contrôle à distance / Debian 11](/acces-locaux-distants-debian11/#12_-_Test_de_connexion), les 3 routes statiques permettant de joindre les VM du réseau local virtuel.

\[C:\\~\] route add -p 192.168.2.0 mask 255.255.255.0 192.168.x.w

\[C:\\~\] route add -p 192.168.3.0 mask 255.255.255.0 192.168.x.w

\[C:\\~\] route add -p 192.168.4.0 mask 255.255.255.0 192.168.x.w

192.168.x.w est l'IP de la carte réseau RED d'IPFire.  
Le \-p déclare les routes comme étant permanentes.

\- Etape 2  
Ajoutez ces 2 lignes au fichier DNS C:\\Windows\\System32\\drivers\\etc\\hosts :

\# loupvirtuel.fr
192.168.4.2 loupvirtuel.fr

\- Etape 3  
Finissez en testant l'URL https://loupvirtuel.fr :

[![Capture - Site Web : Accueil du domaine loupvirtuel.fr](/wp-content/uploads/2023/03/srvdmz-dotclear-430x221.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-dotclear.webp)

Site Web : Accueil du domaine loupvirtuel.fr

Voilà, vous êtes prêt pour installer et utiliser Cockpit.

### 2 - Installation d'un Cockpit primaire sur srvdmz

Cockpit permet l'administration de son système Linux _(Cockpit primaire)_ ainsi que l'administration centralisée d'autres systèmes Linux _(Cockpits secondaires)_.

Les onglets de son interface Web proposent de :

\-- Partie Système --

- Visualiser l'état du matériel -> _Aperçu_

- Lire les journaux système -> _Journaux_

- Visualiser les disques -> _Stockage_

- Visualiser le trafic réseau -> _Réseau_

- Gérer les comptes utilisateurs -> _Comptes_

- Gérer les services -> _Services_

\-- Partie Outils --

- Gérer les applications -> _Applications_

- Gérer les MAJ -> _Mises à jour logicielles_

- Travailler avec un Terminal Web -> _Terminal_

Il est également possible, depuis cette interface, de :

- Gérer d'autres serveurs Linux _(centralisation)_

- Gérer, créer des VM _(plugin)_

- Gérer des conteneurs Podman _(plugin)_

- Etc…

Pour installer Cockpit sur srvdmz, entrez ces Cdes :

\[srvdmz@srvdmz:~$\] su root

\[root@srvdmz:~$\] cd /etc/apt/sources.list.d/

\[root@srvdmz:~$\] echo "deb http://deb.debian.org/debian bullseye-backports main" > backports.list

\[root@srvdmz:~$\] exit

\[srvdmz@srvdmz:~$\] sudo apt update

\[srvdmz@srvdmz:~$\] sudo apt install -t bullseye-backports cockpit

\[srvdmz@srvdmz:~$\] sudo apt install -t bullseye-backports cockpit-pcp

\[srvdmz@srvdmz:~$\] sudo systemctl enable cockpit.socket

\[srvdmz@srvdmz:~$\] sudo systemctl start cockpit

### 3 - Configurations Port/Pare-feu/SSL

#### _3.1 - Port utilisé par Cockpit_

Le numéro de port par défaut de Cockpit est le 9090. Vous allez, par sécurité, modifier celui-ci.

Commencez par créer un dossier cockpit.socket.d :

\[srvdmz@srvdmz:~$\] cd /etc/systemd/system/
\[srvdmz@srvdmz:~$\] sudo mkdir cockpit.socket.d

Générez dans celui-ci un fichier listen.conf :

\[srvdmz@srvdmz:~$\] cd /etc/systemd/system/
\[srvdmz@srvdmz:~$\] sudo nano cockpit.socket.d/listen.conf 

et entrez ceci pour déclarer l'usage du port 9528 :

\[Socket\]
ListenStream=
ListenStream=9528

Pour finir, rechargez la nouvelle configuration systemd et relancez la partie réseau de Cockpit :

\[srvdmz@srvdmz:~$\] sudo systemctl daemon-reload
\[srvdmz@srvdmz:~$\] sudo systemctl restart cockpit.socket

Le Cockpit de srvdmz écoutera ainsi sur le port 9528.

#### _3.2 - Pare-feu_

Autorisez l'usage du port 9528 au niveau de la VM IPFire _(Ref: Mémento [DNS split / Debian 11](/dns-split-debian11/#3_-_Modification_du_pare-feu_IPFire))_ :

[![Capture - IPFire : Utilisation du port 9528 autorisé](/wp-content/uploads/2023/03/ipfire-cockpit-430x235.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/ipfire-cockpit.webp)

IPFire : Utilisation du port 9528 autorisé

#### _3.3 - SSL_

Cockpit fournit de base un certificat SSL auto-signé pour les connexions HTTPS entrantes, une alerte de sécurité est alors affichée par les navigateurs Web.

Vous allez, pour éviter cela, utiliser le certificat du domaine loupvirtuel.fr _(Ref : Mémento [LAMP HTTPS CMS / Debian 11 : Partie 2](/https-dotclear-debian11/#5_-_Mise_en_place_du_protocole_HTTPS))_.

Commencez par copier ces 2 fichiers SSL dans le dossier /etc/cockpit/ws-certs.d :

\[srvdmz@srvdmz:~$\] cd /etc/ssl
  
\[srvdmz@srvdmz:~$\] sudo cp loupvirtuel.crt /etc/cockpit/ws-certs.d 
  
\[srvdmz@srvdmz:~$\] sudo cp loupvirtuel.key /etc/cockpit/ws-certs.d

Renommez le certificat auto-signé de Cockpit :

\[srvdmz@srvdmz:~$\] cd /etc/cockpit/ws-certs.d
  
\[srvdmz@srvdmz:~$\] sudo mv 0-self-signed.cert 0-self-signed.cert-save

et modifiez les droits des 2 fichiers importés :

\[srvdmz@srvdmz:~$\] sudo chown root:cockpit-ws loupvirtuel.\*

Redémarrez Cockpit :

\[srvdmz@srvdmz:~$\] sudo systemctl restart cockpit
\[srvdmz@srvdmz:~$\] sudo systemctl status cockpit

Cockpit est maintenant accessible depuis l'URL :  
https://loupvirtuel.fr:9528

[![Capture - Cockpit : Fenêtre de login](/wp-content/uploads/2023/03/srvdmz-cockpit-login-430x249.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-login.webp)

Cockpit : Fenêtre de login

Connectez-vous en tant qu'utilisateur srvdmz :

[![Capture - Cockpit : Accueil = Onglet Aperçu](/wp-content/uploads/2023/03/srvdmz-cockpit-accueil-430x179.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-accueil.webp)

Cockpit : Accueil = Onglet Aperçu

\-> Bouton Activez l'accès administrateur

Une fenêtre Passer à l’accès administrateur s'ouvre :  
\-> Mot de passe de srvdmz : Entrez le MDP de srvdmz  
\-> Bouton S'authentifier

Le contenu de la zone de notification, soit celui du fichier /etc/motd, peut maintenant être modifié en cliquant sur l'icône d'édition située dans la zone.

Accédez maintenant au Widget Utilisation :  
\-> Voir les métriques et l'historique

Ceux-ci apparaîtront au bout de quelques instants.

_Nota : Si vous disposez d'un nom de domaine routé par votre Box Internet sur l'un de vos serveurs, vous pouvez utiliser celui-ci pour joindre le Cockpit de la VM srvdmz, ceci en créant une règle de proxy inverse appropriée._

### 4 - Interface Web de Cockpit

Observez maintenant le contenu de chacun des modules de Cockpit, soit :

\-- Partie Système --

- Journaux (Priorité = Erreur et au dessus)

[![Capture - Cockpit : Onglet Journaux](/wp-content/uploads/2023/03/srvdmz-cockpit-journaux-430x173.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-journaux.webp)

Cockpit : Onglet Journaux

Contrôle des logs du système, options de filtrage.

- Stockage

[![Capture - Cockpit : Onglet Stockage](/wp-content/uploads/2023/03/srvdmz-cockpit-stockage-430x179.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-stockage.webp)

Cockpit : Onglet Stockage

Gestion du stockage, options de gestion RAID et NFS.

- Réseau

[![Capture - Cockpit : Onglet Réseau](/wp-content/uploads/2023/03/srvdmz-cockpit-reseau-430x177.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-reseau.webp)

Cockpit : Onglet Réseau

Gestion du réseau, options d'ajout de Lien/Pont/VLAN.

- Comptes

[![Capture - Cockpit : Onglet Comptes](/wp-content/uploads/2023/03/srvdmz-cockpit-comptes-430x151.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-comptes.webp)

Cockpit : Onglet Comptes

Gestion des comptes du système, option de création.

- Services

[![Capture - Cockpit : Onglet Services](/wp-content/uploads/2023/03/srvdmz-cockpit-services-430x151.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-services.webp)

Cockpit : Onglet Services

Gestion des statuts des services, options de filtrage.

\-- Partie Outils --

- Applications

[![Capture - Cockpit : Onglet Applications](/wp-content/uploads/2023/03/srvdmz-cockpit-applications-430x157.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-applications.webp)

Cockpit : Onglet Applications

Gestion des extensions utilisées _(plugins)_.

- Mises à jour logicielles

[![Capture - Cockpit : Onglet Mises à jour logicielles](/wp-content/uploads/2023/03/srvdmz-cockpit-maj-430x151.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-maj.webp)

Cockpit : Onglet Mises à jour logicielles

MAJ des paquets du système, option de redémarrage.

- Terminal _(très pratique)_

[![Capture - Cockpit : Onglet Terminal](/wp-content/uploads/2023/03/srvdmz-cockpit-terminal-430x154.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/srvdmz-cockpit-terminal.webp)

Cockpit : Onglet Terminal

Usage de la ligne de Cde pour administrer le système.

La déconnexion de Cockpit s'effectue depuis le menu Session situé en haut et à droite de la page Web.

### 5 - Gestion centralisée des autres VM du réseau

Il est possible de gérer, hormis les conteneurs LXC, l'ensemble des VM du réseau local virtuel depuis le Cockpit installé sur la VM srvdmz.

Pour cela, il faut rendre les VM à gérer accessibles en protocole SSH _(Ref: Mémento [Accès distant SSH sur la VM OpenvSwitch](/acces-locaux-distants-debian11/#5_-_Acces_distant_SSH_sur_la_VM_OpenvSwitch))_ et installer Cockpit sur celles-ci.

SSH ne propose pour l'instant que la lecture seule sur les VM distantes, il faudra toujours cliquer sur le bouton Activez l'accès administrateur de celles-ci pour les gérer.

SSH augmentera la sécurité entre les VM et évitera d'avoir à saisir les MDP de façon répétée.

Les VM à gérer depuis le Cockpit de la VM srvdmz seront à créer en tant que nouveaux hôtes.

#### _5.1 - Ajout de srvlan (premier nouvel hôte)_

Commencez par installer un serveur SSH sur srvlan :

\[srvlan@srvlan:~$\] sudo apt install openssh-server
\[srvlan@srvlan:~$\] sudo systemctl status sshd

Editez ensuite le fichier de configuration de SSH :

\[srvlan@srvlan:~$\] sudo nano /etc/ssh/sshd\_config

et remplacez la ligne #Port 22 par Port 222.

Relancez le service SSH pour traiter la modification :

\[srvlan@srvlan:~$\] sudo systemctl restart sshd

et installez Cockpit sur la VM srvlan comme pratiqué sur la VM srvdmz.

Connectez-vous ensuite sur le Cockpit de srvdmz et cliquez sur le menu déroulant situé en haut et à gauche de la page Web :  
\-> Bouton Ajouter un nouvel hôte

Une fenêtre Ajouter un nouvel hôte s'ouvre :  
\-> Hôte : srvlan.intra.loupvirtuel.fr:222  
\-> Nom d'utilisateur : srvlan  
\-> Couleur : Choisissez une couleur pour la VM  
\-> Bouton Ajouter

Une fenêtre Nouvel hôte s'ouvre :  
\-> Bouton Accepter la clé et se connecter

Une fenêtre Connectez-vous à srvlan@... s'ouvre :  
\-> Mot de passe : MDP de srvlan  
\-> Cochez Créer une nouvelle clé SSH et l'autoriser

Le fenêtre ouverte s'étend :  
\-> Mot de passe clé : MDP de srvdmz et non de srvlan  
\-> Confirmer ... de passe clé : MDP de srvdmz  
\-> Bouton Connexion

L'hôte srvlan@... est ajouté dans le menu déroulant situé en haut et à gauche de la page Web.

Fermez la connexion srvdm@... depuis le menu Session situé en haut et à droite de la page Web.

Remarques :  
\- Un hôte srvlan.intra.loupvirtuel.fr a été ajouté dans le fichier /home/srvdmz/.ssh/known\_hosts.

\- 2 clés SSH id\_rsa et id\_rsa.pub ont été créées sur srvdmz dans /home/srvdmz/.ssh/.

\- La clé id\_rsa.pub a été copiée comme authorized\_keys sur srvlan dans /home/srvlan/.ssh/.

Reconnectez-vous maintenant sur le Cockpit de la VM srvdmz et sélectionnez ensuite l'hôte srvlan@..., la liaison sera directement établie sans demande de MDP.

[![Capture - Cockpit : Gestion de srvlan depuis srvdmz](/wp-content/uploads/2023/03/cockpit-ajout-hote-srvlan-430x190.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/cockpit-ajout-hote-srvlan.webp)

Cockpit : Gestion de la VM srvlan depuis la VM srvdmz

Les connexions futures depuis le Cockpit de la VM srvdmz sur celui de la VM srvlan se feront sans demande de MDP, ceci grâce à la configuration SSH.

#### _5.2 - Ajout d'ovs (second nouvel hôte)_

Un serveur SSH écoute déjà sur le port 222.

Ajoutez seulement Cockpit avec les mêmes Cdes que celles utilisées sur la VM srvdmz.

Connectez-vous ensuite sur le Cockpit de srvdmz et cliquez sur le menu déroulant situé en haut et à gauche de la page Web :  
\-> Bouton Ajouter un nouvel hôte

Une fenêtre Ajouter un nouvel hôte s'ouvre :  
\-> Hôte : ovs.intra.loupvirtuel.fr:222  
\-> Nom d'utilisateur : switch  
\-> Couleur : Choisissez une couleur pour la VM  
\-> Bouton Ajouter

Une fenêtre Nouvel hôte s'ouvre :  
\-> Bouton Accepter la clé et se connecter

L' hôte ovs est alors ajouté dans le fichier :  
/home/srvdmz/.ssh/known\_hosts

Une fenêtre Connectez-vous à switcl@... s'ouvre :  
\-> Mot de passe : MDP de switch  
\-> Cochez cette fois Autoriser la clé SSH  
\-> Bouton Connexion

La clé SSH publique id\_rsa.pub de srvdmz est alors copiée comme authorized\_keys sur ovs.

L'hôte switch@... a été ajouté dans le menu déroulant situé en haut et à gauche de la page Web.

Sélectionnez à présent celui-ci, la connexion doit s'établir directement sans demande de MDP.

Vous savez maintenant comment faire pour ajouter les 2 VM debian11-vm\*.

#### _5.3 - Connexion directe sur Cockpit secondaire_

L'accès direct sur un Cockpit secondaire se fera au travers du Cockpit primaire de la VM srvdmz.

Pour, par exemple, joindre le Cockpit de debian11-vm1 :  
\-> Accédez à la fenêtre de login du Cockpit de srvdmz  
\-> Remplissez ensuite les champs comme ci-dessous

[![Capture - Cockpit : Liaison directe sur un Cockpit secondaire](/wp-content/uploads/2023/03/cockpit-login-direct-cockpit-secondaire-430x347.webp "Cliquez pour agrandir l’image")](/wp-content/uploads/2023/03/cockpit-login-direct-cockpit-secondaire.webp)

Cockpit : Liaison directe sur un Cockpit secondaire

\-> Cliquez sur le bouton Connexion

Une fenêtre Nouvel hôte s'ouvre :  
\-> Bouton Accepter la clé et se connecter

La connexion est établie, l'acceptation de la clé SSH ne sera plus demandé à l'avenir.

### 6 - Bilan

Cockpit est léger, fiable, sécurisé et son interface Web permet d'administrer un système Linux dans son ensemble.

La gestion centralisée de plusieurs systèmes Linux est un plus.

Il propose aussi un peu de supervision, ceci en affichant quelques métriques bien utiles sous forme graphique.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
OK, maintenant si une supervision  
plus poussée vous intéresse, le  
mémento 12.2 vous attend pour  
découvrir l'outil Centreon.

[Mémento 12.2](https://familleleloup.no-ip.org/supervision-centreon-partie-1/)
