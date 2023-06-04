---
title: "Centreon / Debian 11 : 2/6"
date: "2023-04-07"
categories: 
  - "outil-centreon"
---

## Mémento 12.2 - NAS Synology et Qnap

### 2 - Supervision de serveurs NAS

Découvrez d'abord l'interface Web de Centreon ainsi que les éléments supervisés par celle-ci.

#### _2.1 - Interface Web de Centreon_

Menu de gauche :  
Accueil = Vues personnalisées _(aucune de base)_  
Supervision = Etats/Journaux des éléments supervisés  
Rapports = Diagrammes de vue sur des temps donnés  
Configuration = Paramètres des éléments supervisés  
Administration = Gestion de Centreon, des plugins, etc...

Bandeau supérieur - Zone Collecteurs :  
Vert = Le collecteur envoie ses données au central

Bandeau supérieur - Zone Services :  
4 couleurs = Etat Critique/Alerte/Inconnu/OK

Bandeau supérieur - Zone Hôtes :  
3 couleurs = Etat Indisponible/Injoignable/OK

#### _2.2 - Eléments supervisés par Centreon_

Hôte = Elément avec une adresse IP/DNS _(Ex: Serveur)_  
Service = Elément supervisé sur un hôte _(Ex: Cpu)_

Les valeurs de chaque élément sont récupérées par des sondes _(plugins)_ exécutées périodiquement par le collecteur _(Centreon Engine)_.

Le statut des hôtes/services est accessible depuis le menu Supervision -> Statut des ressources.

Des outils appelés modèles d'hôtes et de services ou Plugin Packs facilitent la configuration des hôtes et services à superviser.

La licence Centreon IT-100 permet la découverte automatique d'hôtes et de services.

#### _2.3 - Supervision d'un NAS Synology_

\-- Côté serveur NAS -- 
Activez le serveur SNMP.  
Activez les protocoles SNMPv1 SNMPv2c.  
Nommez votre communauté SNMP _(Ex: snmp14)_.

\-- Côté Plateforme Centreon IT-100 -- 
\-> Menu Configuration > Packs de plugins  
\-> Champ Mots clés > Entrez Synology  
\-> Bouton Recherche

\-> Cliquez sur le + pour installer le plugin trouvé  
\-> Bouton Appliquer

[![Capture - Centreon : Installation du plugin Synology](/wp-content/uploads/2023/03/centreon-ajout-plugin-synology-430x154.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-ajout-plugin-synology.webp)

Centreon : Installation du plugin Synology

[![Capture - Centreon : Plugin Synology installé](/wp-content/uploads/2023/03/centreon-plugin-synology-ok-430x142.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-plugin-synology-ok.webp)

Centreon : Plugin Synology installé

Une fois le plugin installé, créez l'hôte NAS comme suit :  
\-> Menu Configuration > Hôtes > Hôtes  
\-> Bouton Ajouter

Remplissez l'onglet Configuration de l'hôte comme ceci :

[![Capture - Centreon : Informations de base sur l'hôte](/wp-content/uploads/2023/03/centreon-creation-hote-synology-1-430x182.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-creation-hote-synology-1.webp)

Centreon : Informations de base sur l'hôte

[![Capture - Centreon : Options contrôle de l'hôte/ordonnancement](/wp-content/uploads/2023/03/centreon-creation-hote-synology-2-430x171.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-creation-hote-synology-2.webp)

Centreon : Options contrôle de l'hôte/ordonnancement

Remplissez ensuite l'onglet Notification comme ceci :

[![Capture - Centreon : Notification par email désactivée](/wp-content/uploads/2023/03/centreon-creation-hote-synology-3-430x184.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-creation-hote-synology-3.webp)

Centreon : Notification par email désactivée

Ensuite, configurez les services associés à l'hôte :  
\-> Menu Configuration  
\-> Services > Services par l'hôte  
\-> Cliquez sur la roue dentée de chaque service

\-> Onglet Informations générales  
Configurez une période de contrôle

Exemple pour le service Cpu :

[![Capture - Centreon : Période de contrôle du service Cpu configurée ](/wp-content/uploads/2023/03/centreon-service-cpu-periode-controle-430x262.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-service-cpu-periode-controle.webp)

Centreon : Période de contrôle du service Cpu configurée

\-> Onglet Notifications  
Configurez la notification par email à Non

[![Capture - Centreon : Notification par email du service Cpu désactivée](/wp-content/uploads/2023/03/centreon-service-cpu-notifications-non-430x177.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-service-cpu-notifications-non.webp)

Centreon : Notification par email du service Cpu désactivée

sauf pour le service Ping que vous cocherez à Oui

[![Capture - Centreon : Notification par email du service Ping activée](/wp-content/uploads/2023/03/centreon-service-ping-notifications-oui-430x176.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-service-ping-notifications-oui.webp)

Centreon : Notification par email du service Ping activée

Pour finir, déployez la nouvelle configuration :  
\-> Menu Configuration > Collecteurs  
\-> Collecteurs > Cochez le collecteur Central  
\-> Bouton Exporter la configuration

Zone Actions :  
\-> Cochez Déplacer les fichiers générés  
\-> Cochez Redémarrer l'ordonnanceur  
\-> Bouton Exporter

[![Capture - Centreon : Déploiement de la nouvelle configuration](/wp-content/uploads/2023/03/centreon-export-config-ok-430x242.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-export-config-ok.webp)

Centreon : Déploiement de la nouvelle configuration

#### _2.4 - Supervision d'un NAS Qnap_

Installez le plugin Qnap et procédez à la configuration de Centreon comme ci-dessus.

Le tout terminé et fonctionnel, vous pourriez observer sur les services l'état récurrent suivant :

UNKNOWN: SNMP Table Request: Timeout

Pour y remédier, modifiez le timeout des services concernés commme suit :

[![Capture - Centreon : Timeout du service Hardware-Global modifié à 20s](/wp-content/uploads/2023/03/centreon-modification-timeout-430x168.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-modification-timeout.webp)

Centreon : Timeout du service Hardware-Global modifié à 20s

Tout devrait rentrer dans l'ordre.

Pour info, la requête du service Centreon Hardware-Global vers le serveur Qnap est transmise depuis la VM centreon à l'aide de la cde suivante :

\# /usr/lib/centreon/plugins/centreon\_qnap.pl --plugin=storage::qnap::snmp::plugin --mode=hardware --hostname=192.168.9.1 --snmp-version=2c --snmp-community=snmp14 --snmp-timeout 20

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà pour les serveurs NAS.  
La partie 3 vous attend pour  
cette fois traiter la supervision des  
serveurs Debian 11 et Windows 11.

[Partie 3](https://familleleloup.no-ip.org/supervision-centreon-snmp-debian-windows-partie-3/)
