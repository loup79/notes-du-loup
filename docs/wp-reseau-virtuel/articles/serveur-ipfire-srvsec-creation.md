---
title: "srvsec - VBox/IPFire"
date: "2022-02-14"
categories: 
  - "serveur-srvsec"
---

## Mémento 2.11 - Serveur srvsec

Comme pour le réseau sous Debian 10, vous allez créer et placer la VM srvsec _(IPFire)_ en entrée du réseau local ou celle-ci servira entre autres de pare-feu.

### 1 - Construction de la VM depuis VirtualBox

L'utilisation de VirtualBox est considérée acquise.

A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](/virtualbox-installation/)  
[VirtualBox – Mode d’accès réseau par pont](/virtualbox-pont-reseau/)

#### _1.1 - Création et configuration_

Téléchargez l'ISO x86\_64 version 2.x - Core Update y :  
[https://www.ipfire.org/download](https://www.ipfire.org/download)

Démarrez l'application VirtualBox puis :

Menu Nouvelle de VirtualBox :  
\- Nom IPFire - Type Linux - Version Other ... (64-bit)  
\- Taille de la mémoire > 512 Mo  
\- Disque dur > Créer un disque dur virtuel maintenant  
\- Type de fichier de disque dur > VDI  
\- Stockage sur disque dur ... > Dynamiquement alloué  
\- Emplacement du fichier et taille > 8 Go > Créer

La VM est créée dans le panneau gauche de VirtualBox.

Sélectionnez la nouvelle VM, puis :

Menu Configuration de VirtualBox :  
\- - - Onglet Système  
\> Carte mère > Ordre d'amorçage > Décochez Disquette  
\> Carte mère > Fonctions avancées > Cochez IO-APIC  
\> Processeur > 2 CPU et cochez PAE/NX  
  
\- - - Onglet Stockage  
\> Unités de stockage > Sélectionnez Vide  
\> Attributs > Cliquez sur l'icône CD  
\> Sélectionnez Choisissez un fichier de disque ...  
\> Entrez le chemin de l'image ISO IPFire > Ouvrir

\- - - Onglet Réseau  
\> Carte 1  
\> Mode d'accès réseau > Accès par pont  
\> Nom > Sélectionnez la carte réseau active du PC hôte  
\> Avancé > Notez l'adresse MAC dans un coin

\> Carte 2 > Cochez Activer la carte réseau  
\> Mode d'accès réseau > Réseau interne  
\> Avancé > Notez l'adresse MAC dans un coin

\> Carte 3 > Cochez Activer la carte réseau  
\> Mode d'accès réseau > Réseau interne  
\> Avancé > Notez l'adresse MAC dans un coin  
  
\> OK

Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de la distribution IPFire_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

Menu Démarrer de VirtualBox :  
La VM créée s'exécute.

![Capture - IPFire : Début Installation](/wp-content/uploads/2018/03/Ipfire_1-430x317.png)

Installation IPFire - Image 1

Validez Install IPFire 2.x - Core y > Touche Entrée

![Capture - IPFire : Choix de la langue](/wp-content/uploads/2018/03/Ipfire_2.png)

Installation IPFire - Image 2

Sélectionnez le Français > OK

![Capture - IPFire : Démarrage de l'installation](/wp-content/uploads/2018/03/Ipfire_3.png)

Installation IPFire - Image 3

Validez Démarrer l'installation > Touche Entrée

![Capture - IPFire : Acceptation de la licence](/wp-content/uploads/2018/03/Ipfire_4-430x242.png)

Installation IPFire - Image 4

Cochez J'accepte la licence > OK

![Capture - IPFire : Préparation du disque dur](/wp-content/uploads/2018/03/Ipfire_5.png)

Installation IPFire - Image 5

Validez Supprime toutes les données > Touche Entrée

![Capture - IPFire : Choix du système de fichiers](/wp-content/uploads/2018/03/Ipfire_6.png)

Installation IPFire - Image 6

Validez Système de fichier ext4 > OK  
L'installation du système commence ...

![Capture - IPFire : Redémarrage](/wp-content/uploads/2018/03/Ipfire_7.png)

Installation IPFire - Image 7

Validez Redémarrer > Touche Entrée

![Capture - IPFire : Choix de la langue du clavier](/wp-content/uploads/2018/03/Ipfire_8.png)

Installation IPFire - Image 8

Suite au redémarrage, la fenêtre ci-dessus s'affiche.  
Sélectionnez le type de clavier fr > OK

![Capture - IPFire : Choix du fuseau horaire](/wp-content/uploads/2018/03/Ipfire_9.png)

Installation IPFire - Image 9

Sélectionnez le fuseau horaire Europe/Paris > OK

![Capture - IPFire : Nom d'hôte](/wp-content/uploads/2018/03/Ipfire_10.png)

Installation IPFire - Image 10

Entrez le nom d'hôte de la VM soit srvsec > OK

![Capture - IPFire : Nom de domaine](/wp-content/uploads/2018/03/Ipfire_11.png)

Installation IPFire - Image 11

Entrez le nom de domaine loupipfire.fr > OK  
C'est le nom qui sera exploité plus tard sur le réseau.

![Capture - IPFire : Mot de passe utilisateur root](/wp-content/uploads/2018/03/Ipfire_12-430x160.png)

Installation IPFire - Image 12

Entrez le MDP pour root _(2 fois)_ > OK

![Capture - IPFire : Mot de passe utilisateur admin](/wp-content/uploads/2018/03/Ipfire_13.png)

Installation IPFire - Image 13

Entrez celui de l'administrateur Web admin _(2fois)_ > OK

![Capture - IPFire : Choix type de configuration réseau](/wp-content/uploads/2018/03/Ipfire_14.png)

Installation IPFire - Image 14

Sélectionnez Type de configuration réseau > OK

![Capture - IPFire : Sélection réseau green+red+orange](/wp-content/uploads/2018/03/Ipfire_15.png)

Installation IPFire - Image 15

Choix de GREEN_(lan)_ + RED_(wan)_ + ORANGE_(dmz)_ > OK

![Capture - IPFire : Choix affectation des pilotes et cartes](/wp-content/uploads/2018/03/Ipfire_16.png)

Installation IPFire - Image 16

Sélectionnez Affectation des Pilotes et des Cartes > OK

![Capture - IPFire : Sélection carte green](/wp-content/uploads/2018/03/Ipfire_17.png)

Installation IPFire - Image 17

\> GREEN _\=_ carte 3 de VirtualBox > Sélectionner

![Capture - IPFire : Sélection carte réseau 3](/wp-content/uploads/2018/03/Ipfire_18-430x165.png)

Installation IPFire - Image 18

Selon son adresse MAC > carte 3 > Sélectionner

![Capture - IPFire : Carte 3 affectée à green](/wp-content/uploads/2018/03/Ipfire_19.png)

Installation IPFire - Image 19

La carte 3 est bien affectée à la zone GREEN.

![Capture - IPFire : Sélection carte red](/wp-content/uploads/2018/03/Ipfire_20.png)

Installation IPFire - Image 20

\> RED _\=_ carte 1 de VirtualBox > Sélectionner

![Capture - IPFire : Sélection carte réseau 1 ](/wp-content/uploads/2018/03/Ipfire_21-430x141.png)

Installation IPFire - Image 21

Selon son adresse MAC > carte 1 > Sélectionner

![Capture - IPFire : Sélection carte orange](/wp-content/uploads/2018/03/Ipfire_22.png)

Installation IPFire - Image 22

\> ORANGE _\=_ carte 2 de VirtualBox > Sélectionner

![Capture - IPFire : Sélection carte réseau 2 ](/wp-content/uploads/2018/03/Ipfire_23-430x141.png)

Installation IPFire - Image 23

On ne peut pas se tromper > Sélectionner

![Capture - IPFire : Vue attribution des cartes réseau](/wp-content/uploads/2018/03/Ipfire_24.png)

Installation IPFire - Image 24

Les 3 cartes réseaux sont configurées > Terminé

![Capture - IPFire : Configuration d'adresse](/wp-content/uploads/2018/03/Ipfire_25.png)

Installation IPFire - Image 25

Sélectionnez Configuration d'adresse \> OK

![Capture - IPFire : Sélection carte green](/wp-content/uploads/2018/03/Ipfire_26.png)

Installation IPFire - Image 26

\> GREEN soit la carte réseau 3 de VirtualBox > OK

![Capture - IPFire : Message d'alerte](/wp-content/uploads/2018/03/Ipfire_27.png)

Installation IPFire - Image 27

Petit avertissement > OK

![Capture - IPFire : Adresse IP carte green](/wp-content/uploads/2018/03/Ipfire_28.png)

Installation IPFire - Image 28

Entrez l'adresse IP de la carte 3 soit 192.168.2.1 > OK

![Capture - IPFire : Sélection carte orange](/wp-content/uploads/2018/03/Ipfire_29.png)

Installation IPFire - Image 29

\> ORANGE soit la carte réseau 2 de VirtualBox > OK

![Capture - IPFire : Adresse IP carte orange](/wp-content/uploads/2018/03/Ipfire_30.png)

Installation IPFire - Image 30

Entrez l'adresse IP de la carte 2 soit 192.168.4.1 > OK

![Capture - IPFire : Sélection carte red](/wp-content/uploads/2018/03/Ipfire_31.png)

Installation IPFire - Image 31

\> RED soit la carte réseau 1 de VirtualBox > OK

![Capture - IPFire : Adresse IP carte red](/wp-content/uploads/2018/03/Ipfire_32.png)

Installation IPFire - Image 32

\- Statique > Vérifiez que la case est bien cochée  
\- Adresse IP > IP libre du serveur DHCP de votre Box  
\- Gateway _(Selon version IPFire)_ > IP de votre Box > OK

![Capture - IPFire : Fin de la configuration des adresses IP](/wp-content/uploads/2018/03/Ipfire_33.png)

Installation IPFire - Image 33

Les 3 cartes ont maintenant une adresse IP > Terminé

![Capture - IPFire : Choix réglages dns et passerelle](/wp-content/uploads/2018/03/Ipfire_34.png)

Installation IPFire - Image 34

Selon version IPFire > Réglages DNS et Passerelle \> OK  
Sinon > Terminé

![Capture - IPFire : Réglages dns et passerelle](/wp-content/uploads/2018/03/Ipfire_35.png)

Installation IPFire - Image 35

\- DNS pri../sec... > IP DNS enregistrées dans votre Box  
\- Passerelle par défaut > IP de votre Box > OK

![Capture - IPFire : Fin réglages dns et passerelle](/wp-content/uploads/2018/03/Ipfire_36.png)

Installation IPFire - Image 36

C'est fini pour cette partie > Terminé

![Capture - IPFire : Configuration dhcp](/wp-content/uploads/2018/03/Ipfire_37.png)

Installation IPFire - Image 37

Cette fenêtre s'affiche, ne pas activer le DHCP > OK

![Capture - IPFire : Fin du paramétrage](/wp-content/uploads/2018/03/Ipfire_38.png)

Installation IPFire - Image 38

Et voilà, c'est terminé > OK

IPFire finit son démarrage et affiche le login de srvsec :

[![Capture - IPFire : Premier démarrage](https://familleleloup.no-ip.org/wp-content/uploads/2018/03/Ipfire_39-430x322.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2018/03/Ipfire_39.png)

IPFire : Premier démarrage

Vous pouvez à présent vous connecter en tant que root.

Notez qu'IPFire peut être reconfiguré avec la Cde setup :

\[root@srvsec:~$\] setup

Arrêtez maintenant la VM srvsec :

\[root@srvsec:~$\] poweroff             \# ou Cde shutdown -h now

Retirez, si nécessaire, l'image ISO du lecteur CD virtuel.  
Pour cela, Menu Configuration de VirtualBox :  
\- - - Onglet Stockage  
\> Unités de stockage > ipfire-...iso  
\> Attributs > Cliquez sur l'icône CD  
\> Retirer le disque du lecteur virtuel > OK

Redémarrez et vérifiez que tout se passe bien.

#### _1.3 - Connexion sur l'interface WEB d'IPFire_

Ouvrez le navigateur Web de la VM srvlan et entrez l'adresse https://192.168.2.1:444.

Un message d'alerte de sécurité s'affiche :  
\- Si navigateur Chromium  
\> Paramètres avancés > Continuer vers le site ...  
\- Si navigateur Firefox  
\> Avancé... > Accepter le risque et poursuivre

Une fenêtre de connexion s'ouvre :  
\> Nom d'utilisateur > admin  
\> Mot de passe > Votre MDP créé ci-dessus Image 13  
\> Connexion ou OK

La page d'accueil d'IPFire doit s'afficher :

[![Capture - IPFire : Page d'accueil Web](/wp-content/uploads/2018/03/Ipfire_40-430x192.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2018/03/Ipfire_40.png)

IPFire : Page d'accueil Web

#### _1.4 - Route statique vers le réseau 192.168.3.0_

Vérifiez depuis srvsec qu'un ping sur l'IP 192.168.3.1 de srvlan ne fonctionne pas actuellement.

Pour corriger cela, revenez sur srvlan :  
\> Page d'accueil IPFire > Réseau > Routes statiques

Remplissez les champs comme montré ci-dessous et cliquez ensuite sur Ajouter :

[![Capture - IPFire : Ajout d'une route statique vers 192.168.3.0](/wp-content/uploads/2018/03/Ipfire_ajout_route_statique-430x242.png "Cliquez pour agrandir l'image")](/wp-content/uploads/2018/03/Ipfire_ajout_route_statique.png)

IPFire : Ajout d'une route statique vers 192.168.3.0

Le ping doit maintenant recevoir une réponse positive.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà !  
IPFire est en place. Le mémento  
3.11 vous attend pour la création  
du dernier serveur soit srvdmz.

[Mémento 3.11](https://familleleloup.no-ip.org/serveur-debian11-srvdmz-creation/)
