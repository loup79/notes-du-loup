---
title: "srvsec - VBox/IPFire"
date: "2023-07-14"
categories: 
  - "serveur-srvsec"
---

## Mémento 2.1 - Serveur srvsec

Comme pour les réseaux précédents, vous allez créer et placer la VM srvsec _(IPFire)_ en entrée du réseau local ou celle-ci servira entre autres de pare-feu.

### 1 - Construction de la VM depuis VirtualBox

L'utilisation de VirtualBox est considérée acquise.

A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](../virtualbox-installation/)  
[VirtualBox – Mode d’accès réseau par pont](../virtualbox-pont-reseau/)

#### _1.1 - Création et configuration_

Téléchargez l'ISO x86\_64 version 2.x - Core Update y :  
[https://www.ipfire.org/download](https://www.ipfire.org/download)

\- Démarrez ensuite l'application VirtualBox 7.x, puis :  
\- - Menu de VirtualBox > Machine > Nouvelle...  
\-> Nom : IPFire  
\-> Folder : Sélectionnez le dossier de stockage des VM  
\-> ISO Image : Sélectionnez l'ISO téléchargée ci-dessus  
\-> Type : Other  
\-> Version : Other/Unknown (64-bit)  
\-> Bouton Suivant

\-> Mémoire vive : 512 MB  
\-> Processors : 2 CPU si possible  
\-> Bouton Suivant

\-> Create a Virtual Hard Disk Now : Ajustez à 8 Go  
\-> Bouton Suivant

\-> Vérifiez le Récapitulatif  
\-> Bouton Finish

La VM s'affiche dans le panneau gauche de VirtualBox.

\- Sélectionnez maintenant la nouvelle VM, puis :  
\- - Menu de VirtualBox > Machine > Configuration...    
\- - - Onglet Système    
\-> Carte mère > Ordre d'amorçage > Décochez Disquette    
\-> Carte mère > Fonctions avancées > Cochez IO-APIC    
\-> Processeur > Cochez PAE/NX  
  
\- - - Onglet Réseau  
\-> Adapter 1 > Mode d'accès réseau > Accès par pont  
\-> Name > Indiquez la carte réseau active du PC hôte  
\-> Advanced > Notez l'adresse MAC dans un coin

\-> Adapter 2 > Cochez Activer la carte réseau  
\-> Mode d'accès réseau > Réseau interne  
\-> Advanced > Notez l'adresse MAC dans un coin

\-> Adapter 3 > Cochez Activer la carte réseau  
\-> Mode d'accès réseau > Réseau interne  
\-> Advanced > Notez l'adresse MAC dans un coin  
  
\> OK

Les autres paramètres peuvent rester inchangés.

#### _1.2 - Installation de la distribution IPFire_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

\- - Menu de VirtualBox > Machine > Démarrer  
\-> Démarrage normal _(La VM s'exécute)_

![Capture - IPFire : Accueil Installation](../wp-content/uploads/2023/07/IPFire_1-430x319.webp)

Installation IPFire - Image 1

Validez Install IPFire 2.x - Core y > Touche Entrée

![Capture - IPFire : Sélection de la langue](../wp-content/uploads/2023/07/IPFire_2.webp)

Installation IPFire - Image 2

Sélectionnez le Français > OK

![Capture - IPFire : Validation du démarrage de l'installation](../wp-content/uploads/2023/07/IPFire_3.webp)

Installation IPFire - Image 3

Validez Démarrer l'installation > Touche Entrée

![Capture - IPFire : Acceptation de la licence](../wp-content/uploads/2023/07/IPFire_4-430x243.webp)

Installation IPFire - Image 4

Cochez J'accepte la licence > OK

![Capture - IPFire : Préparation du disque dur](../wp-content/uploads/2023/07/IPFire_5.webp)

Installation IPFire - Image 5

Validez Supprime toutes les données > Touche Entrée

![Capture - IPFire : Sélection du système de fichiers ext4](../wp-content/uploads/2023/07/IPFire_6.webp)

Installation IPFire - Image 6

Sélectionnez le système de fichier ext4 > OK  
L'installation du système commence ...

![Capture - IPFire : Validation de la demande de redémarrage](../wp-content/uploads/2023/07/IPFire_7-430x228.webp)

Installation IPFire - Image 7

Validez Redémarrer > Touche Entrée

![Capture - IPFire : Sélection de la langue du clavier](../wp-content/uploads/2023/07/IPFire_8.webp)

Installation IPFire - Image 8

Suite au redémarrage, la fenêtre ci-dessus s'affiche.  
Sélectionnez le type de clavier fr > OK

![Capture - IPFire : Sélection du fuseau horaire](../wp-content/uploads/2023/07/IPFire_9.webp)

Installation IPFire - Image 9

Sélectionnez le fuseau horaire Europe/Paris > OK

![Capture - IPFire : Entrée du nom d'hôte](../wp-content/uploads/2023/07/IPFire_10.webp)

Installation IPFire - Image 10

Entrez le nom d'hôte de la VM soit srvsec > OK

![Capture - IPFire : Entrée du nom de domaine](../wp-content/uploads/2023/07/IPFire_11.webp)

Installation IPFire - Image 11

Entrez le nom de domaine loupipfire.fr > OK  
C'est le nom qui sera exploité plus tard sur le réseau.

![Capture - IPFire : Entrée du mot de passe utilisateur root](../wp-content/uploads/2023/07/IPFire_12-430x172.webp)

Installation IPFire - Image 12

Entrez le MDP pour root _(2 fois)_ > OK

![Capture - IPFire : Entrée du mot de passe utilisateur admin](../wp-content/uploads/2023/07/IPFire_13-430x170.webp)

Installation IPFire - Image 13

Entrez celui de l'administrateur Web admin _(2 fois)_ > OK

![Capture - IPFire : Sélection type de configuration réseau](../wp-content/uploads/2023/07/IPFire_14.webp)

Installation IPFire - Image 14

Sélectionnez Type de configuration réseau > OK

![Capture - IPFire : Sélection du tpe de réseau green+red+orange](../wp-content/uploads/2023/07/IPFire_15.webp)

Installation IPFire - Image 15

Choix de GREEN_(lan)_ + RED_(wan)_ + ORANGE_(dmz)_ > OK

![Capture - IPFire : Choix affectation des pilotes et cartes](../wp-content/uploads/2023/07/IPFire_16.webp)

Installation IPFire - Image 16

Sélectionnez Affectation des Pilotes et des Cartes > OK

![Capture - IPFire : Sélection carte réseau green](../wp-content/uploads/2023/07/IPFire_17-430x222.webp)

Installation IPFire - Image 17

\-> GREEN soit la carte 3 de VirtualBox > Sélectionner

![Capture - IPFire : Sélection adresse MAC de la carte réseau 3 de VirtualBox](../wp-content/uploads/2023/07/IPFire_18-430x165.webp)

Installation IPFire - Image 18

Accédez à l'adresse MAC de la carte 3 > Sélectionner

![Capture - IPFire : Carte 3 de VirtualBox affectée à green](../wp-content/uploads/2023/07/IPFire_19-430x247.webp)

Installation IPFire - Image 19

La carte 3 est bien affectée à la zone GREEN.

![Capture - IPFire : Sélection carte réseau red](../wp-content/uploads/2023/07/IPFire_20-430x246.webp)

Installation IPFire - Image 20

\-> RED soit la carte 1 de VirtualBox > Sélectionner

![Capture - IPFire : Sélection adresse MAC de la carte réseau 1 de VirtualBox ](../wp-content/uploads/2023/07/IPFire_21-430x153.webp)

Installation IPFire - Image 21

Accédez à l'adresse MAC de la carte 1 > Sélectionner

![Capture - IPFire : Sélection carte réseau orange](../wp-content/uploads/2023/07/IPFire_22-430x272.webp)

Installation IPFire - Image 22

\-> ORANGE soit la carte 2 de VirtualBox > Sélectionner

![Capture - IPFire : Sélection adresse MAC de la carte réseau 2 de VirtualBox](../wp-content/uploads/2023/07/IPFire_23-430x142.webp)

Installation IPFire - Image 23

On ne peut pas se tromper > Sélectionner

![Capture - IPFire : Vue attribution des cartes réseau](../wp-content/uploads/2023/07/IPFire_24-430x295.webp)

Installation IPFire - Image 24

Les 3 cartes réseaux sont configurées > Terminé

![Capture - IPFire : Sélection configuration d'adresse](../wp-content/uploads/2023/07/IPFire_25.webp)

Installation IPFire - Image 25

Sélectionnez Configuration d'adresse \> OK

![Capture - IPFire : Configuration d'adresse, sélection carte green](../wp-content/uploads/2023/07/IPFire_26-430x211.webp)

Installation IPFire - Image 26

\-> GREEN soit la carte réseau 3 de VirtualBox > OK

![Capture - IPFire : Configuration d'adresse, avertissement](../wp-content/uploads/2023/07/IPFire_27-430x202.webp)

Installation IPFire - Image 27

Petit avertissement > OK

![Capture - IPFire : Adresse IP carte green](../wp-content/uploads/2023/07/IPFire_28.webp)

Installation IPFire - Image 28

Entrez l'adresse IP de la carte 3 soit 192.168.2.1 > OK

![Capture - IPFire : Configuration d'adresse, sélection carte orange](../wp-content/uploads/2023/07/IPFire_29-430x212.webp)

Installation IPFire - Image 29

\-> ORANGE soit la carte réseau 2 de VirtualBox > OK

![Capture - IPFire : Adresse IP carte orange](../wp-content/uploads/2023/07/IPFire_30.webp)

Installation IPFire - Image 30

Entrez l'adresse IP de la carte 2 soit 192.168.4.1 > OK

![Capture - IPFire : Configuration d'adresse, sélection carte red](../wp-content/uploads/2023/07/IPFire_31-430x211.webp)

Installation IPFire - Image 31

\-> RED soit la carte réseau 1 de VirtualBox > OK

![Capture - IPFire : Adresse IP carte red](../wp-content/uploads/2023/07/IPFire_32.webp)

Installation IPFire - Image 32

\- Statique : Vérifiez que la case est bien cochée  
\- Adresse IP : IP libre du serveur DHCP de votre Box  
\- Gateway : IP de votre Box > OK

![Capture - IPFire : Fin de la sélection des cartes réseau green, orange et red](../wp-content/uploads/2023/07/IPFire_33-430x211.webp)

Installation IPFire - Image 33

Les 3 cartes ont maintenant une adresse IP > Terminé

![Capture - IPFire : Configuration d'adresse, sortie](../wp-content/uploads/2023/07/IPFire_34.webp)

Installation IPFire - Image 34

Quittez la Configuration d'adresse > Terminé

![Capture - IPFire : Configuration dhcp](../wp-content/uploads/2023/07/IPFire_37-430x300.webp)

Installation IPFire - Image 35

Cette fenêtre s'affiche, ne pas activer le DHCP > OK

![Capture - IPFire : Installation terminée](../wp-content/uploads/2023/07/IPFire_38.webp)

Installation IPFire - Image 36

Et voilà, c'est terminé > OK

IPFire finit son démarrage et affiche le login de srvsec :

[![Capture - IPFire : Premier démarrage](../wp-content/uploads/2023/07/IPFire_39-430x251.webp "Cliquez pour agrandir l'image")](../wp-content/uploads/2023/07/IPFire_39.webp)

IPFire : Premier démarrage

Vous pouvez à présent vous connecter en tant que root.

Notez qu'IPFire peut être reconfiguré avec la Cde setup :

```bash
[root@srvsec:~$\] setup
```
Arrêtez maintenant la VM srvsec :

```bash
[root@srvsec:~$\] poweroff    # ou Cde shutdown -h now
```

Retirez, si nécessaire, l'image ISO du lecteur CD virtuel :  
\- - Menu de VirtualBox > Machine > Configuration…  
\- - - Onglet Stockage  
\-> Zone Unités de stockage > ipfire-...iso  
\-> Zone Attributs > Cliquez sur l'icône CD  
\-> Retirer le disque du lecteur virtuel > OK

Redémarrez et vérifiez que tout se passe bien.

#### _1.3 - Connexion sur l'interface WEB d'IPFire_

Ouvrez le navigateur Web de la VM srvlan et entrez l'adresse https://192.168.2.1:444.

Un message d'alerte de sécurité s'affiche :  
\-> Bouton Avancé...  
\-> Bouton Accepter le risque et poursuivre
  
Une fenêtre de connexion s'ouvre :  
\-> Nom d'utilisateur > Entrez admin  
\-> Mot de passe > Entrez votre MDP créé en Image 13  
\-> Bouton Connexion

La page d'accueil d'IPFire doit s'afficher :

[![Capture - IPFire : Page d'accueil Web](../wp-content/uploads/2023/07/IPFire_accueil_web-430x208.webp "Cliquez pour agrandir l'image")](../wp-content/uploads/2023/07/IPFire_accueil_web.webp)

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

[Mémento 3.11](/notes-du-loup/wp-reseau-virtuel/articles/serveur-debian11-srvdmz-creation/)
