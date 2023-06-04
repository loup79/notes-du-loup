---
title: "Centreon / Debian 11 : 4/6"
date: "2023-04-04"
categories: 
  - "outil-centreon"
---

## Mémento 12.2 - Auto-découverte

### 5 - Centreon Auto Discovery

Le module de découverte automatique inclus dans la licence Centreon IT-100 permet de détecter de nouveaux hôtes ou services via le protocole SNMP.

Attention, les hôtes ne seront découverts que si un agent snmp est installé sur chacun d'eux.

En ce qui me concerne, 5 agents ont été répartis sur 1 serveur, 3 VM et 1 conteneur LXD, tous sous Debian 11 et dans la communauté snmp14.

Commencez donc par installer vos agents en vous référant à la [Partie 3](/supervision-centreon-snmp-debian-windows-partie-3/#32_-_Installation_et_configuration_de_SNMP_sur_Debian).

Vérifiez ensuite que le [module](/wp-content/uploads/2023/03/Centreon-module-decouverte-auto.pdf) Auto Discovery fourni avec la [licence](/wp-content/uploads/2023/03/Centreon-IT100-licence.pdf) IT-100 est bien installé.

A défaut, ajoutez le module comme suit :

sudo apt install centreon-auto-discovery-server

Puis ouvrez l'interface Web de Centreon :  
\-> Menu Administration  
\-> Extensions > Gestionnaire  
\-> Installez le module Auto Discovery

#### _5.1 - Ajout du plugin Generic SNMP_

Vous utiliserez, pour détecter les nouveaux hôtes, le plugin Generic SNMP qui fait partie des plugins permettant l'auto-découverte, voir la [docs.centreon.com](/wp-content/uploads/2023/03/Centreon-pack-generic-SNMP.pdf).

Pour cela, commencez par installer ce paquet :

sudo apt install centreon-plugin-applications-protocol-snmp

puis le plugin en vous référant au [§ 2.3](/supervision-centreon-nas-partie-2/#23_-_Supervision_dun_NAS_Synology) de la Partie 2.

Relancez par précaution les services de Centreon :

sudo systemctl restart cbd centengine gorgoned

#### _5.2 - Création d'une tâche d'auto-découverte_

Créez à présent une tâche d'auto-découverte depuis l'interface Web comme ci-dessous :  
\-> Configuration > Hôtes  
\-> Découverte > Bouton _\+ AJOUTER_

Remplir la partie 1 comme suit :

[![Capture - Centreon : Auto-découverte - Partie 1/6](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-1-430x302.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-1.webp)

Centreon : Auto-découverte - Partie 1/6

Laissez la partie 2 comme telle :

[![Capture - Centreon : Auto-découverte - Partie 2/6](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-2-430x300.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-2.webp)

Centreon : Auto-découverte - Partie 2/6

Remplir la partie 3 comme suit :

[![Capture - Centreon : Auto-découverte - Partie 3/6](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-3-430x304.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-3.webp)

Centreon : Auto-découverte - Partie 3/6

Laissez la partie 4 comme telle :

[![Capture - Centreon : Auto-découverte - Partie 4/6](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-4-430x300.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-4.webp)

Centreon : Auto-découverte - Partie 4/6

Remplir la partie 5 comme suit :

[![Capture - Centreon : Auto-découverte - Partie 5/6](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-5-430x304.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-5.webp)

Centreon : Auto-découverte - Partie 5/6

Remplir la partie 6 comme suit :

[![Capture - Centreon : Auto-découverte - Partie 6/6](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-6-430x304.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-6.webp)

Centreon : Auto-découverte - Partie 6/6

La tâche est exécutée et son statut apparaît dans :  
\-> Configuration > Hôtes > Découverte

[![Capture - Centreon : Tâche Auto découverte SNMP terminée](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-7-430x97.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/centreon-autodiscovery-7.webp)

Centreon : Tâche Auto découverte SNMP terminée

La découverte des 5 hôtes a pris environ 5 minutes.

#### _5.3 - Exploitation du résultat de la tâche_

Lorsque le statut indique terminé, cliquez sur la flèche située à droite de celui-ci pour voir le détail du résultat de l'analyse manuelle.

Sélectionnez ensuite les hôtes que vous souhaitez ajouter ou mettre à jour dans la configuration de Centreon, puis cliquez sur l'icône Sauvegarder.

Pour ma part, le serveur, les 3 VM et le conteneur LXD ont bien été découverts.

Gérez enfin les périodes d'ordonnancement ainsi que les notifications des hôtes découverts et leurs services associés en vous aidant du [§ 2.3](/supervision-centreon-nas-partie-2/#23_-_Supervision_dun_NAS_Synology) de la Partie 2.

Une fois fait, déployez la nouvelle configuration :  
\-> Menu Configuration > Collecteurs  
\-> Collecteurs > Cochez le collecteur Central  
\-> Bouton Exporter la configuration

Zone Actions, cochez :  
Générer les fichiers de configuration  
Lancer le débogage du moteur de supervision (-v)  
Déplacer les fichiers générés  
Redémarrer l'ordonnanceur

Cliquez ensuite sur le bouton Exporter.

#### _5.4 - Activation/Désactivation d'un service_

Vérifiez après quelques minutes les statuts des hôtes et des services, personnellement, un seul état Critique concernant la ressource Swap de l'une de mes 3 VM.

Statut correct car la VM en question ne dispose pas d'une partition Swap.

Pour supprimer un service inutile, désactiver celui-ci comme suit et redéployer la configuration :

[![Capture - Centreon : Désactivation du service swap](/wp-content/uploads/2023/03/centreon-desactivation-service-1-430x139.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-desactivation-service-1.webp)

Centreon : Désactivation du service swap

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Belle étape. La partie 5 vous  
attend pour découvrir comment  
superviser un site Web en utilisant  
le plugin HTTP Server.

[Partie 5](https://familleleloup.no-ip.org/supervision-centreon-curl-sites-web-partie-5/)
