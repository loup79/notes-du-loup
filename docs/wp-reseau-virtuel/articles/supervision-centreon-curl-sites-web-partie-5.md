---
title: "Centreon / Debian 11 : 5/6"
date: "2023-04-02"
categories: 
  - "outil-centreon"
---

## Mémento 12.2 - Sites Web avec Curl

### 6 - Supervision d'un site Web avec Curl et Ping

La supervision d’un site Web réalisée avec les Cdes Curl et Ping permettra de vérifier la disponibilité et les performances de celui-ci.

Les graphiques de Centreon afficheront les temps de réponse HTTP issus de la Cde curl et les temps de réponse et paquets perdus issus de la Cde ping.

La Cde curl permet de se connecter sur différents types de serveurs et de communiquer avec ceux-ci via différents types de protocoles dont le HTTPS, FTP, etc…

La partie ci-dessous s'inspire de la [docs.centreon.com](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/Centreon-applications-protocol-http.pdf).

#### _6.1 - Installation des plugins HTTP de Centreon_

Pour surveiller l’URL d'un site, procédez comme suit :  
\-- Côté VM centreon --

sudo apt install centreon-plugin-applications-protocol-http

\-- Côté Plateforme Centreon IT-100 -- 
Menu Configuration  
\-> Packs de plugins  
\-> Champ Mots clés > Entrez HTTP  
\-> Bouton Recherche

\-> Sélectionnez dans le résultat le plugin HTTP Server  
\-> Cliquez sur son icône + pour lancer son installation

Le plugin apparaît installé après quelques secondes.

#### _6.2 - Configuration du site Web (hôte Web)_

Créez un nouvel hôte Web comme suit :  
Menu Configuration  
\-> Hôtes > Hôtes  
\-> Bouton Ajouter

Remplissez l'onglet Configuration de l'hôte.

[![Capture - Centreon : Information de base sur l'hôte Web](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-hote-site-web-430x200.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/04/centreon-hote-site-web.webp)

Centreon : Information de base sur l'hôte Web

Choisissez un site dont vous connaissez l'URL _(Adresse IPv4 / FQDN)_ et le port HTTPS utilisés.

Pour éviter d'éventuels soucis de DNS, ajoutez ces 2 lignes dans le fichier /etc/hosts de la VM centreon seulement si le central de Centreon et le site Web se trouvent sur le même LAN :

Ex: IP locale du site Web       Ex: URL du site
192.168.7.42                           monblog.fr

Remplissez ensuite l'onglet Notification.

[![Capture - Centreon : Notification par e-mail pour l'hôte à Non](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-hote-siteweb-notification-430x169.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-hote-siteweb-notification.webp)

Centreon : Notification par e-mail pour l'hôte à Non

Puis, configurez les 2 services associés à l'hôte Web :  
Menu Configuration  
\-> Services > Services par l'hôte

\-> Affichez le service HTTP-Response-Time de l'hôte  
\-> Onglet Informations générales

Modifiez la valeur URLPATH dans le cas d'une URL de type https://domaine/sous-dossier/.

[![Capture - Centreon : URL de type https://domaine/sous-dossier/](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-service-siteweb-url-sous-dossier-430x181.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-service-siteweb-url-sous-dossier.webp)

Centreon : URL type https://domaine/sous-dossier/

\-> Onglet Notifications

Configurez la notification par e-mail à Non.

[![Capture - Centreon : Notification HTTP-Response-Time  à Non](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-service-siteweb-notifications-430x66.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-service-siteweb-notifications.webp)

Centreon : Notification HTTP-Response-Time à Non

\-> Affichez ensuite le service Ping de l'hôte Web  
\-> Onglet Notifications

Configurez la notification par e-mail à Oui.

[![Capture - Centreon : Notification Ping  à Oui](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-service-ping-notifications-oui-430x176.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-service-ping-notifications-oui.webp)

Centreon : Notification Ping à Oui

#### _6.3 - Activation du site Web_

Pour activer l'hôte Web, déployez la configuration :  
Menu Configuration  
\-> Collecteurs > Collecteurs  
\-> Cochez le collecteur Central  
\-> Bouton Exporter la configuration

\-- Zone Actions -- 
\-> Cochez Déplacer les fichiers générés  
\-> Cochez Redémarrer l'ordonnanceur  
\-> Bouton Exporter

![Capture - Centreon : Configuration de l'hôte Web déployée](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-siteweb-deploiement.webp)

Centreon : Configuration de l'hôte Web déployée

#### _6.4 - Statistiques du site Web_

Pour afficher les graphiques du site, procédez ainsi :  
Menu Statut des ressources  
\-> Champ Rechercher > Entrez le filtre type:host  
\-> Colonne G _(Graphique)_  
\-> Cliquez sur l'icône Service graphs de l'hôte Web

[![Capture - Centreon : Statistiques graphiques d'un site Web](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-hote-siteweb-statistique-430x236.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-hote-siteweb-statistique.webp)

Centreon : Statistiques de l'hôte Web sur 7 jours

#### _6.5 - Duplication d'hôte / Test de la Cde HTTP Server_

\-- Duplication -- 
Pour créer un nouvel hôte Web, utilisez la duplication :  
Menu Configuration  
\-> Hôtes > Hôtes  
\-> Cochez le nom de l'hôte à dupliquer  
\-> Champ Plus d'actions > Sélectionnez Dupliquer  
\-> Confirmez la duplication.

Modifiez ensuite uniquement les paramètres propres au nouveau site Web.

\-- Cde du plugin HTTP Server -- 
Testez par curiosité, depuis la VM centreon, la Cde HTTP Server pour le site google.fr :

sudo /usr/lib/centreon/plugins/centreon\_protocol\_http.pl \\
    --plugin=apps::protocols::http::plugin \\
    --mode=response \\
    --hostname=google.fr \\
    --proto='https' \\
    --port='443' \\
    --urlpath='/' \\
    --warning=' ' \\
    --critical=' '

Retour normal :

```
OK: response time 0.534s | 'time'=0.534s;;;0; 'size'=14051B;;;0;
```

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Plutôt facile. La partie 6 vous  
attend pour superviser une base  
de données MySQL et créer une  
vue personnalisée Centreon.

[Partie 6](https://familleleloup.no-ip.org/supervision-centreon-bdd-mysql-partie-6/)
