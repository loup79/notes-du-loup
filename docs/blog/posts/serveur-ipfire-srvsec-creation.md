---
title: "srvsec - VBox/IPFire"
summary: Création d'une VM pare-feu WAN de nom srvsec.
authors: 
  - G.Leloup
date: 2023-07-14
categories: 
  - 2 - Serveur WAN "srvsec"
---

<figure markdown>
  ![Image - Firewall ou pare-feu](../images/2023/07/firewall.webp){ width="430" }
</figure>

## Mémento 2.1 - Serveur srvsec

Comme pour les réseaux précédents, vous allez créer et placer la VM srvsec _(IPFire)_ en entrée du réseau local ou celle-ci servira entre autres de pare-feu.

### Construction de la VM

L'utilisation de VirtualBox est considérée acquise.

A défaut, référez-vous aux mémentos suivants :  
[VirtualBox - Installation](../posts/virtualbox-installation.md){ target="_blank" }  
[VirtualBox - Mode d’accès réseau par pont](../posts/virtualbox-pont-reseau.md){ target="_blank" }

#### _- Création et configuration_

Téléchargez l'ISO x86\_64 version 2.x - Core Update y :  
[https://www.ipfire.org/download](https://www.ipfire.org/download){ target="_blank" }

<!-- more -->

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

#### _- Installation d'IPFire_

Conseil pratique avant de démarrer la nouvelle VM :  
Si le curseur de la souris disparaît lors d'un clic dans la fenêtre de la VM, celui-ci peut être récupéré par le PC hôte à l'aide de la touche CTRL située à droite de la barre d'espace du clavier.

\- - Menu de VirtualBox > Machine > Démarrer  
\-> Démarrage normal _(La VM s'exécute)_

<figure markdown>
  ![Capture - IPFire : Accueil Installation](../images/2023/07/IPFire_1.webp){ width="430" }
  <figcaption>Installation IPFire - Image 1</figcaption>
</figure>

Validez Install IPFire 2.x - Core y > Touche Entrée

<figure markdown>
  ![Capture - IPFire : Sélection de la langue](../images/2023/07/IPFire_2.webp)
  <figcaption>Installation IPFire - Image 2</figcaption>
</figure>

Sélectionnez le Français > OK

<figure markdown>
  ![Capture - IPFire : Validation du démarrage de l'installation](../images/2023/07/IPFire_3.webp)
  <figcaption>Installation IPFire - Image 3</figcaption>
</figure>

Validez Démarrer l'installation > Touche Entrée

<figure markdown>
  ![Capture - IPFire : Acceptation de la licence](../images/2023/07/IPFire_4.webp){ width="430" }
  <figcaption>Installation IPFire - Image 4</figcaption>
</figure>

Cochez J'accepte la licence > OK

<figure markdown>
  ![Capture - IPFire : Préparation du disque dur](../images/2023/07/IPFire_5.webp)
  <figcaption>Installation IPFire - Image 5</figcaption>
</figure>

Validez Supprime toutes les données > Touche Entrée

<figure markdown>
  ![Capture - IPFire : Sélection du système de fichiers ext4](../images/2023/07/IPFire_6.webp)
  <figcaption>Installation IPFire - Image 6</figcaption>
</figure>

Sélectionnez le système de fichier ext4 > OK  
L'installation du système commence ...

<figure markdown>
  ![Capture - IPFire : Validation de la demande de redémarrage](../images/2023/07/IPFire_7.webp){ width="430" }
  <figcaption>Installation IPFire - Image 7</figcaption>
</figure>

Validez Redémarrer > Touche Entrée

<figure markdown>
  ![Capture - IPFire : Sélection de la langue du clavier](../images/2023/07/IPFire_8.webp)
  <figcaption>Installation IPFire - Image 8</figcaption>
</figure>

Suite au redémarrage, la fenêtre ci-dessus s'affiche.  
Sélectionnez le type de clavier fr > OK

<figure markdown>
  ![Capture - IPFire : Sélection du fuseau horaire](../images/2023/07/IPFire_9.webp)
  <figcaption>Installation IPFire - Image 9</figcaption>
</figure>

Sélectionnez le fuseau horaire Europe/Paris > OK

<figure markdown>
  ![Capture - IPFire : Entrée du nom d'hôte](../images/2023/07/IPFire_10.webp)
  <figcaption>Installation IPFire - Image 10</figcaption>
</figure>

Entrez le nom d'hôte de la VM soit srvsec > OK

<figure markdown>
  ![Capture - IPFire : Entrée du nom de domaine](../images/2023/07/IPFire_11.webp)
  <figcaption>Installation IPFire - Image 11</figcaption>
</figure>

Entrez le nom de domaine loupipfire.fr > OK  
C'est le nom qui sera exploité plus tard sur le réseau.

<figure markdown>
  ![Capture - IPFire : Entrée du mot de passe utilisateur root](../images/2023/07/IPFire_12.webp){ width="430" }
  <figcaption>Installation IPFire - Image 12</figcaption>
</figure>

Entrez le MDP pour root _(2 fois)_ > OK

<figure markdown>
  ![Capture - IPFire : Entrée du mot de passe utilisateur admin](../images/2023/07/IPFire_13.webp){ width="430" }
  <figcaption>Installation IPFire - Image 13</figcaption>
</figure>

Entrez celui de l'administrateur Web admin _(2 fois)_ > OK

<figure markdown>
  ![Capture - IPFire : Sélection type de configuration réseau](../images/2023/07/IPFire_14.webp)
  <figcaption>Installation IPFire - Image 14</figcaption>
</figure>

Sélectionnez Type de configuration réseau > OK

<figure markdown>
  ![Capture - IPFire : Sélection du tpe de réseau green+red+orange](../images/2023/07/IPFire_15.webp)
  <figcaption>Installation IPFire - Image 15</figcaption>
</figure>

Choix de GREEN_(lan)_ + RED_(wan)_ + ORANGE_(dmz)_ > OK

<figure markdown>
  ![Capture - IPFire : Choix affectation des pilotes et cartes](../images/2023/07/IPFire_16.webp)
  <figcaption>Installation IPFire - Image 16</figcaption>
</figure>

Sélectionnez Affectation des Pilotes et des Cartes > OK

<figure markdown>
  ![Capture - IPFire : Sélection carte réseau green](../images/2023/07/IPFire_17.webp){ width="430" }
  <figcaption>Installation IPFire - Image 17</figcaption>
</figure>

\-> GREEN soit la carte 3 de VirtualBox > Sélectionner

<figure markdown>
  ![Capture - IPFire : Sélection adresse MAC de la carte réseau 3 de VirtualBox](../images/2023/07/IPFire_18.webp){ width="430" }
  <figcaption>Installation IPFire - Image 18</figcaption>
</figure>

Accédez à l'adresse MAC de la carte 3 > Sélectionner

<figure markdown>
  ![Capture - IPFire : Carte 3 de VirtualBox affectée à green](../images/2023/07/IPFire_19.webp){ width="430" }
  <figcaption>Installation IPFire - Image 19</figcaption>
</figure>

La carte 3 est bien affectée à la zone GREEN.

<figure markdown>
  ![Capture - IPFire : Sélection carte réseau red](../images/2023/07/IPFire_20.webp){ width="430" }
  <figcaption>Installation IPFire - Image 20</figcaption>
</figure>

\-> RED soit la carte 1 de VirtualBox > Sélectionner

<figure markdown>
  ![Capture - IPFire : Sélection adresse MAC de la carte réseau 1 de VirtualBox](../images/2023/07/IPFire_21.webp){ width="430" }
  <figcaption>Installation IPFire - Image 21</figcaption>
</figure>

Accédez à l'adresse MAC de la carte 1 > Sélectionner

<figure markdown>
  ![Capture - IPFire : Sélection carte réseau orange](../images/2023/07/IPFire_22.webp){ width="430" }
  <figcaption>Installation IPFire - Image 22</figcaption>
</figure>

\-> ORANGE soit la carte 2 de VirtualBox > Sélectionner

<figure markdown>
  ![Capture - IPFire : Sélection adresse MAC de la carte réseau 2 de VirtualBox](../images/2023/07/IPFire_23.webp){ width="430" }
  <figcaption>Installation IPFire - Image 23</figcaption>
</figure>

On ne peut pas se tromper > Sélectionner

<figure markdown>
  ![Capture - IPFire : Vue attribution des cartes réseau](../images/2023/07/IPFire_24.webp){ width="430" }
  <figcaption>Installation IPFire - Image 24</figcaption>
</figure>

Les 3 cartes réseaux sont configurées > Terminé

<figure markdown>
  ![Capture - IPFire : Sélection configuration d'adresse](../images/2023/07/IPFire_25.webp)
  <figcaption>Installation IPFire - Image 25</figcaption>
</figure>

Sélectionnez Configuration d'adresse \> OK

<figure markdown>
  ![Capture - IPFire : Configuration d'adresse, sélection carte green](../images/2023/07/IPFire_26.webp){ width="430" }
  <figcaption>Installation IPFire - Image 26</figcaption>
</figure>

\-> GREEN soit la carte réseau 3 de VirtualBox > OK

<figure markdown>
  ![Capture - IPFire : Configuration d'adresse, avertissement](../images/2023/07/IPFire_27.webp){ width="430" }
  <figcaption>Installation IPFire - Image 27</figcaption>
</figure>

Petit avertissement > OK

<figure markdown>
  ![Capture - IPFire : Adresse IP carte green](../images/2023/07/IPFire_28.webp)
  <figcaption>Installation IPFire - Image 28</figcaption>
</figure>

Entrez l'adresse IP de la carte 3 soit 192.168.2.1 > OK

<figure markdown>
  ![Capture - IPFire : Configuration d'adresse, sélection carte orange](../images/2023/07/IPFire_29.webp){ width="430" }
  <figcaption>Installation IPFire - Image 29</figcaption>
</figure>

\-> ORANGE soit la carte réseau 2 de VirtualBox > OK

<figure markdown>
  ![Capture - IPFire : Adresse IP carte orange](../images/2023/07/IPFire_30.webp)
  <figcaption>Installation IPFire - Image 30</figcaption>
</figure>

Entrez l'adresse IP de la carte 2 soit 192.168.4.1 > OK

<figure markdown>
  ![Capture - IPFire : Configuration d'adresse, sélection carte red](../images/2023/07/IPFire_31.webp){ width="430" }
  <figcaption>Installation IPFire - Image 31</figcaption>
</figure>

\-> RED soit la carte réseau 1 de VirtualBox > OK

<figure markdown>
  ![Capture - IPFire : Adresse IP carte red](../images/2023/07/IPFire_32.webp)
  <figcaption>Installation IPFire - Image 32</figcaption>
</figure>

\- Statique : Vérifiez que la case est bien cochée  
\- Adresse IP : IP libre du serveur DHCP de votre Box  
\- Gateway : IP de votre Box > OK

<figure markdown>
  ![Capture - IPFire : Fin de la sélection des cartes réseau green, orange et red](../images/2023/07/IPFire_33.webp){ width="430" }
  <figcaption>Installation IPFire - Image 33</figcaption>
</figure>

Les 3 cartes ont maintenant une adresse IP > Terminé

<figure markdown>
  ![Capture - IPFire : Configuration d'adresse, sortie](../images/2023/07/IPFire_34.webp)
  <figcaption>Installation IPFire - Image 34</figcaption>
</figure>

Quittez la Configuration d'adresse > Terminé

<figure markdown>
  ![Capture - IPFire : Configuration dhcp](../images/2023/07/IPFire_37.webp){ width="430" }
  <figcaption>Installation IPFire - Image 35</figcaption>
</figure>

Cette fenêtre s'affiche, ne pas activer le DHCP > OK

<figure markdown>
  ![Capture - IPFire : Installation terminée](../images/2023/07/IPFire_38.webp)
  <figcaption>Installation IPFire - Image 36</figcaption>
</figure>

Et voilà, c'est terminé > OK

IPFire finit son démarrage et affiche le login de srvsec :

<figure markdown>
  ![Capture - IPFire : Premier démarrage](../images/2023/07/IPFire_39.webp){ width="430" }
  <figcaption>IPFire : Premier démarrage</figcaption>
</figure>

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

#### _- Accès à l'interface WEB_

Ouvrez le navigateur Web de la VM srvlan et entrez l'adresse `https://192.168.2.1:444`.

Un message d'alerte de sécurité s'affiche :  
\-> Bouton Avancé...  
\-> Bouton Accepter le risque et poursuivre
  
Une fenêtre de connexion s'ouvre :  
\-> Nom d'utilisateur > Entrez admin  
\-> Mot de passe > Entrez votre MDP créé en Image 13  
\-> Bouton Connexion

La page d'accueil d'IPFire doit s'afficher :

<figure markdown>
  ![Capture - IPFire : Page d'accueil Web](../images/2023/07/IPFire_accueil_web.webp){ width="430" }
  <figcaption>IPFire : Page d'accueil Web</figcaption>
</figure>

#### _- Ajout d'une route statique_

Vérifiez depuis srvsec qu'un ping sur l'IP 192.168.3.1 de srvlan ne fonctionne pas actuellement.

Pour corriger cela, revenez sur srvlan :  
\-> Page d'accueil IPFire > Réseau > Routes statiques

Remplissez les champs comme montré ci-dessous et cliquez ensuite sur le bouton Ajouter :

<figure markdown>
  ![Capture - IPFire : Ajout d'une route statique vers 192.168.3.0](../images/2023/07/IPFire_ajout_route_statique.webp){ width="430" }
  <figcaption>IPFire : Ajout d'une route statique vers 192.168.3.0</figcaption>
</figure>

Le ping doit maintenant recevoir une réponse positive.

![Image - Rédacteur satisfait](../images/2023/07/redacteur_satisfait.jpg "Image Pixabay - Mohamed Hassan"){ align=left }

&nbsp;  
Voilà !  
IPFire est en place. Le mémento  
3.1 vous attend pour la création  
du dernier serveur soit srvdmz.

[Mémento 3.1](../posts/serveur-debian12-srvdmz-creation.md){ .md-button .md-button--primary }
