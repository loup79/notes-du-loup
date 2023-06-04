---
title: "Centreon / Debian 11 : 6/6"
date: "2023-04-01"
categories: 
  - "outil-centreon"
---

## Mémento 12.2 - MySQL et Widgets vues

### 7 - Supervision d'une base de données _(Bdd)_ MySQL

L'exemple ci-dessous concerne une Bdd MySQL/MariaDB située sur un serveur Debian :

- Nom de la Bdd = cmswordpress

- IP du collecteur Centreon = 192.168.8.32

La supervision nécessite d'ajouter au sein de la Bdd cmswordpress un utilisateur MySQL dédié au collecteur de Centreon IT-100.

La partie ci-dessous s'inspire de la [docs.centreon.com](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/Centreon-supervision-mysql-mariadb.pdf).

#### _7.1 - Ajout sur la Bdd d'un utilisateur dédié Centreon_

Connectez-vous sur le serveur supportant la Bdd :

sudo mysql -u root -p
\[sudo\] Mot de passe de ... :  
Enter password : Votre MDP root pour MySQL/MariaDB 
MariaDB \[(none)\]>

Listez les Bdd existantes :

MariaDB \[(none)\]> SHOW DATABASES;

La liste des Bdd présentes sur le serveur s'affiche.  
Sélectionnez celle portant le nom cmswordpress :

MariaDB \[(none)\]> USE cmswordpress;   

Database changed  
MariaDB \[cmswordpress\]>

et créez l'utilisateur MySQL dédié Centreon comme suit :

MariaDB \[cmswordpress> CREATE USER 'centreon'@'192.168.8.32' IDENTIFIED BY 'votre-mdp';  
Query OK, 0 rows affected (0,006 sec)

MariaDB \[cmswordpress\]>

Affectez lui ses droits _(privilèges)_ :

MariaDB \[cmswordpress\]> GRANT SELECT ON cmswordpress.\* TO 'centreon'@'192.168.8.32';
Query OK, 0 rows affected (0,004 sec)

MariaDB \[cmswordpress\]>

Terminez en fermant la connexion MySQL :

MariaDB \[cmswordpress\]> quit
Bye

Ci-après, l'utilisateur MySQL de nom centreon vu par un gestionnaire de Bdd tel phpMyAdmin :

[![Capture - phpMyAdmin : Edition de l'utilisateur centreon](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-mariadb-utilisateur-centreon-430x199.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-mariadb-utilisateur-centreon.webp)

phpMyAdmin : Edition de l'utilisateur centreon

#### _7.2 - Installation des plugins MySQL de Centreon_

Pour surveiller une Bdd, procédez comme suit :  
\-- Côté VM centreon -- 
\-> Installez ce paquet Debian si manquant

sudo apt install centreon-plugin-applications-databases-mysql

\-- Côté Plateforme Centreon IT-100 -- 
\-> Menu Configuration > Packs de plugins  
\-> Champ Mots clés > Entrez mysql  
\-> Bouton Recherche  
  
\-> Sélectionnez le plugin titré MySQL/MariaDB  
\-> Cliquez sur son icône + pour lancer son installation  
  
Le plugin apparaît installé après quelques secondes.

#### _7.3 - Configuration de la Bdd sur Centreon_

Créez un nouvel hôte Bdd comme suit :  
\-> Menu Configuration > Hôtes > Hôtes  
\-> Bouton Ajouter

Remplissez l'onglet Configuration de l'hôte :

[![Capture - Centreon : Configuration Bdd cmswordpress](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-hote-bdd-mariadb-430x201.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-hote-bdd-mariadb.webp)

Centreon : Configuration Bdd cmswordpress

Ensuite, comme pour l'hôte Web du [§ 6.2 de Centreon / Debian 11 : 5/6](https://familleleloup.no-ip.org/supervision-centreon-curl-sites-web-partie-5/#62_-_Configuration_du_site_Web_hote_Web), désactivez la notification par e-mail depuis l'onglet Notification.

Concernant les 8 services associés à l'hôte Bdd, référez-vous de nouveau au [§ 6.2](https://familleleloup.no-ip.org/supervision-centreon-curl-sites-web-partie-5/#62_-_Configuration_du_site_Web_hote_Web) pour configurer la notification par e-mail à Non sauf pour le service Ping.

#### _7.4 - Activation de la configuration Bdd_

Pour activer l'hôte Bdd, déployez la configuration :  
\-> Menu Configuration > Collecteurs  
\-> Collecteurs > Cochez le collecteur Central  
\-> Bouton Exporter la configuration  
  
Zone Actions :  
\-> Cochez Déplacer les fichiers générés  
\-> Cochez Redémarrer l'ordonnanceur  
\-> Bouton Exporter  
  
Retour normal :

![Capture - Centreon : Configuration de l'hôte Bdd déployée](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-siteweb-deploiement.webp)

Centreon : Configuration de l'hôte Bdd déployée

#### _7.5 - Statistiques de la Bdd_

Exemple de graphique issu du service Queries :

[![Capture - Centreon : Statistiques graphiques d'une Bdd MySQL](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-supervision-bdd-430x222.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-supervision-bdd.webp)

Centreon : Hôte Bdd, service Queries _(Requêtes)_

### 8 - Création d'une vue personnalisée liée à la Bdd

Les widgets fournis avec Centreon permettent de créer des vues graphiques personnalisées.

Pour créez une première vue, procédez ainsi :  
\-> Menu Accueil > Vues personnalisées  
\-> Cliquez sur l'icône d'édition située en haut et à droite  
Une barre de boutons s'affiche.  
  
\-> Bouton Ajouter une vue  
Une fenêtre Créer une vue s'ouvre.  
  
\-> Champ Nom > Entrez Bdd cmswordpress  
\-> Cochez une mise en page sur 2 colonnes  
\-> Cochez Public pour partager la vue  
\-> Bouton Soumettre  
La vue titrée en onglet cmswordpress s'ouvre vide.

#### _8.1 - Ajout d'un widget dédié au service Bdd Queries_

\-> Bouton Ajouter widget  
Une fenêtre Ajouter widget s'ouvre.  
  
\-> Champ Titre > Entrez Service Queries  
\-> Champ Widget > Sélectionnez Graph Monitoring  
\-> Bouton Soumettre  
Le widget titré Service Queries s'ouvre.  
  
\-> Cliquez sur l'icône clé à molette du widget  
Une fenêtre Widget ... \[graph-monitoring\] s'ouvre.  
  
\-> Champ Service  
\-> a) Sélectionnez l'hôte Debian\_Bdd\_cmswordpress  
\-> b) Sélectionnez le service Queries  
\-> Champ Graph period  
\-> Sélectionnez Last 7 days  
\-> Bouton Apply  
  
Le graphique du widget s'affiche sur la page.

#### _8.2 - Ajout de widgets supplémentaires_

Ajoutez maintenant dans la vue personnalisée de nom Bdd cmswordpress, ceci en vous aidant de la procédure ci-dessus, les widgets dédiés aux services suivants :

- Myisam-Keycache _(Cache du moteur MyISAM)_

- Database-Size _(Taille de la Bdd cmswordpress)_

- Open-Files _(Nombre de fichiers ouverts)_

Résultat :

[![Capture - Centreon : Vue personnalisée d'une Bdd MySQL](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-bdd-vue-personnalisee-430x204.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-bdd-vue-personnalisee.webp)

Centreon : Vue personnalisée d'une Bdd MySQL

![Image - Rédacteur satisfait](https://familleleloup.no-ip.org/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà, la présentation de Centreon  
IT-100 s'achève ici. Vous devriez à  
présent pouvoir découvrir par vous  
même toutes les possibilités du produit.
