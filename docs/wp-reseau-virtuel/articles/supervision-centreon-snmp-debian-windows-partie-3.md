---
title: "Centreon / Debian 11 : 3/6"
date: "2023-04-06"
categories: 
  - "outil-centreon"
---

## Mémento 12.2 - PC Debian et Windows

### 3 - Supervision d'un serveur Debian 11

_Nota : Dans le cas d'un service SNMP déjà présent et installé depuis une Debian antérieure à la 11.6, ajoutez le dépôt non-free dans le fichier /etc/apt/sources.list._

Exemple d'un fichier sources.list modifié :

deb http://deb.debian.org/debian/ bullseye main non-free

deb http://deb.debian.org/debian-security/ bullseye-security main non-free

deb http://deb.debian.org/debian/ bullseye-updates main non-free

L'ajout de ce dépôt est nécessaire si le fichier snmpd.conf est resté configuré d'origine.

Une fois ajouté, installez le paquet non-free suivant :

sudo apt install snmp-mibs-downloader

#### _3.1 - Notes générales sur le protocole SNMP_

SNMP = Simple Network Management Protocol  
MIB = Management Information Base  
OID = Object IDentifier

Le protocole SNMP s’appuie sur un manager et des agents, ces derniers permettant de récupérer des informations _(valeurs)_ sur différents objets.

Les MIBs sont exploitées par le SNMP pour accéder aux informations. Chaque objet SNMP _(périphérique ou élément d'OS)_ dispose d’une MIB.

Les OIDs de forme numérique 1.3.6.1.2... désignent l’emplacement des informations _(valeurs)_ à consulter dans une MIB.

Cdes SNMP de base émises par le manager :  
GET = demande d'une valeur à un agent.  
GET-NEXT = demande de la valeur suivante.  
GET-BULK = demande groupée de valeurs.  
SET = modifie une valeur contenue dans un OID.

Cdes SNMP de base émises par l'agent :  
GET-RESPONSE = répond à GET ou SET.  
TRAP = envoi d'une notification au manager.

Debian utilise les applications suivantes pour lancer les Cdes ci-dessus :

- snmpget

- snmpwalk

- snmpbulkget

- snmpset

- snmptrap

#### _3.2 - Installation/Réglage de SNMP sur Debian_

La partie ci-dessous s'inspire de la [docs.centreon.com](/wp-content/uploads/2023/03/Centreon-supervision-serveur-linux.pdf)[.](/wp-content/uploads/2023/03/Centreon-supervision-serveur-linux.pdf)

Paramètres utilisés pour l'exemple :  
Nom de la communauté SNMP = snmp14  
IP du serveur Debian 11 = 192.168.9.3

Commencez par installer ces paquets sur le serveur :

sudo apt install snmp snmpd libsnmp-perl

Puis, éditez le fichier de configuration snmpd.conf :

sudo nano /etc/snmp/snmpd.conf

et modifiez son contenu comme suit :

agentaddress udp:161
rocommunity snmp14 default \# ou snmp14 192.168.9.0/24
#rocommunity6 ...
view    systemonly    included   .1.3.6.1.2.1.1
view    systemonly    included   .1.3.6.1.2.1.25.1

Par prudence, gardez une seule ligne d'instruction rocommunity, commentez les autres.

Démarrez enfin le service et vérifiez son statut :

sudo systemctl start snmpd
sudo systemctl status snmpd

Si statut Ok, activez le service au boot du système :

sudo systemctl enable snmpd

Vérifiez par curiosité l'ouverture du port SNMP 161 :

ss -ulnp

Le retour affiché doit inclure une ligne comme celle-ci :

UNCONN   0   0   0.0.0.0:161   0.0.0.0:\*

Pour finir, vérifiez le bon fonctionnement de SNMP :

snmpwalk -c snmp14 -v 2c 192.168.9.3

La liste des OIDs de l'agent SNMP doit s'afficher.

#### _3.3 - Réglage côté Centreon_

Installez le plugin Linux SNMP et configurez l'hôte Debian et les services associés en utilisant cette fois le modèle OS-Linux-SNMP-custom.

Vous pouvez vous aider de l'exemple du NAS Synology pour y arriver (ref : [Centreon - Partie 2](/supervision-centreon-nas-partie-2/#23_-_Supervision_dun_NAS_Synology)).

N'oubliez pas pour terminer de déployer la nouvelle configuration.

### 4 - Supervision d'un serveur Windows 11

Comme pour Debian, il est nécessaire d'installer un agent SNMP sur le serveur Windows.

La partie ci-dessous s'inspire de la [docs.centreon.com](/wp-content/uploads/2023/03/Centreon-supervision-serveur-windows.pdf)[.](/wp-content/uploads/2023/03/Centreon-supervision-serveur-linux.pdf)

#### _4.1 - Installation de l'agent SNMP_

\-> Touches Windows + I pour ouvrir les paramètres  
\-> Applications > Fonctionnalités facultatives  
\-> Bouton Afficher les fonctionnalités

Une fenêtre Ajouter une fonctionnalité ... s'ouvre :  
\-> Champ Recherche > Entrez snmp  
\-> Cochez Protocole SNMP _(Simple Network ...)_  
\-> Bouton Suivant > Bouton Installer

L'installation peut durer de 2 à 4 minutes.

#### _4.2 - Réglage du service SNMP_

\-> Touches Windows + R pour ouvrir la fenêtre Exécuter  
\-> Entrez services.msc > OK

Une fenêtre Services s'ouvre :  
Double-cliquez sur le service Service SNMP

Une fenêtre Propriétés de Service SNMP s'ouvre :  
\-- Onglet Sécurité -- 
\-> Ajoutez la communauté snmp14  
\-> Ajoutez l'IP du manager Centreon

\-- Onglet Agent -- 
\-> Rubrique Service  
\-> Cochez les données collectées par Centreon

\-> Bouton Appliquer > OK  
\-> Redémarrer le service  
\-> Fermez la fenêtre Services

#### _4.3 - Réglage côté Centreon_

Installez le plugin Windows SNMP et configurez l'hôte Windows 11 et les services associés en utilisant cette fois le modèle OS-Windows-SNMP-custom.

Vous pouvez vous aider de l'exemple du NAS Synology pour y arriver (ref : [Centreon - Partie 2](/supervision-centreon-nas-partie-2/#23_-_Supervision_dun_NAS_Synology)).

N'oubliez pas pour terminer de déployer la nouvelle configuration.

Vérifiez après quelques minutes le statut de l'hôte et des services associés, personnellement, un seul statut Alerte concernant la ressource Memory.

Alerte normale après vérification de la taille mémoire utilisée par le serveur Windows.

Pour contourner l'alerte, il a fallu modifier le seuil de celle-ci et redéployer la configuration :

[![Capture - Centreon : Modification du seuil d'alerte du service Memory](/wp-content/uploads/2023/03/centreon-modification-valeur-warning-430x172.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-modification-valeur-warning.webp)

Centreon : Modification du seuil d'alerte du service Memory

#### _4.4 - Découverte de nouveaux services_

Le plugin Windows SNMP a permis, lors de la création de l'hôte Windows 11, d'activer automatiquement les services de base suivants : Cpu, Memory, Ping et Swap.

Les plugins peuvent inclure des modèles de services autres que ceux configurés de base.

Pour découvrir ceux du plugin Windows SNMP, procédez comme suit :

[![Capture - Centreon : Ajout du service disponible Disk-C](/wp-content/uploads/2023/03/centreon-decouverte-service-430x111.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2023/03/centreon-decouverte-service.webp)

Centreon : Ajout du service disponible Disk-C

Les services sélectionnés sont alors automatiquement ajoutés à l'hôte Windows 11.

Configurez ensuite chacun d'eux et redéployez la nouvelle configuration.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
On avance. La partie 4 vous  
attend pour exploiter le module  
de découverte automatique  
d'hôtes et de services.

[Partie 4](https://familleleloup.no-ip.org/supervision-centreon-snmp-autodecouverte-partie-4)
