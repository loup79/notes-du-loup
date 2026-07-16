---
title: "NAS - Synology et Home Assistant"  
description: Installation de Home Assistant sur le Synology.  
authors:   
  - G.Leloup  
date: 2025-06-29 
categories:  
  - NAS
---

## Home Assistant (HA)

**Août 2023.**

### Installation

Créer au préalable sur le Synology un utilisateur qui sera dédié à Home Assistant (HA).

Puis télécharger l'image disque _haos_ova-x.y.vmdk.zip_ depuis :  
[https://github.com/home-assistant/operating-system/releases/](https://github.com/home-assistant/operating-system/releases/){ target = "_blank" }.

Créer ensuite une VM Home-Assistant depuis l'application Virtual Machine Manager en utilisant l'image téléchargée.

Dans les paramètres de la VM, penser à déclarer le micrologiciel en UEFI.

Une fois la VM créée et **H**ome **A**ssistant installé _(environ 15mn)_, tester l'URL suivante :  
`http://IP-VM-**HA**:8123`

### Accès HTTPS (Reverse Proxy)

\- Côté Synology, créer une règle de proxy inversé de type :  
`HTTPS://nom-du-domaine:port` vers `HTTP://IP-VM-**HA**:8123`.

Ne pas oublier dans la règle de gérer l'en-tête personnalisé en créant les valeurs du WebSocket.

\- Côté HA, installer le plugin complémentaire File Editor depuis le menu Paramètres -> Applications.

Editer ensuite le fichier configuration.yaml depuis celui-ci et entrer le contenu suivant :  

```markdown
homeassistant:
external_url: "https://nom-du-domaine" # ne pas indiquer le port
internal_url: "http://192.168.x.y:8123" # IP locale de HA avec le port
http:
use_x_forwarded_for: true
trusted_proxies: # IP locale Synology
– 192.168.x.z
ip_ban_enabled: false
```

Vérifier la configuration et si OK _(vert)_ redémarrer HA.

Vérifier pour finir dans HA -> Paramètres -> Système -> Réseau que l'accès externe est activé pour la partie Réseau.

### HACS

Voir les PDF suivants concernant l'installation du **H**ome **A**ssistant **C**ommunity **S**tore :  
[Home-assistant-installer-HACS.pdf](../medias/home-assistant-doc-pdf/Home-assistant-installer-HACS-2024.pdf){ target = "_blank" }

[Home-assistant-HACS-meteo-france.pdf](../medias/home-assistant-doc-pdf/Home-assistant-HACS-meteo-france.pdf){ target = "_blank" }

<!-- more -->

### Intégration Local Tuya

Installer sur la VM supervision _(U820)_ les paquets nodejs et npm.

Installer ensuite l'outil tuya-cli avec la Cde npm :

```bash
sudo npm i @tuyapi/cli -g
```

Puis lancer le wizard depuis l'outl :

```bash
tuya-cli wizard
```

Répondre aux questions du wizard pour au final obtenir la "Local Key" dont a besoin l'intégration Local Tuya pour fonctionner dans Home Assistant.

### Erreur de setup

Message d'erreur affiché :

```markdown
Error returned from Supervisor: System is not ready with state : setup
```

-- Solution --  
Afficher la VM Home Assistant en mode console depuis l'outil Virtual Machine Manager du Synology et entrer la Cde suivante :

```bash
ha > banner
```

### Caméras Heden ONVIF

Cartes HA directions et positions prédéfinies :

-- Configurartion de la carte HA directions --

```yaml
type: glance
title: Caméra du salon
entities:
  - entity: camera.camera_onvif_du_salon_profile0
    tap_action:
      action: call-service
      service: onvif.ptz
      service_data:
        entity_id: camera.camera_onvif_du_salon_profile0
        pan: LEFT
        speed: 1
        distance: 0.3
        move_mode: ContinuousMove
    name: Gauche
    show_state: false
    icon: mdi:arrow-left
  - entity: camera.camera_onvif_du_salon_profile0
    tap_action:
      action: call-service
      service: onvif.ptz
      service_data:
        entity_id: camera.camera_onvif_du_salon_profile0
        pan: RIGHT
        speed: 1
        distance: 0.3
        move_mode: ContinuousMove
    name: Droite
    show_state: false
    icon: mdi:arrow-right
  - entity: camera.camera_onvif_du_salon_profile0
    tap_action:
      action: call-service
      service: onvif.ptz
      service_data:
        entity_id: camera.camera_onvif_du_salon_profile0
        tilt: UP
        speed: 1
        distance: 0.3
        move_mode: ContinuousMove
    name: Haut
    show_state: false
    icon: mdi:arrow-up
  - entity: camera.camera_onvif_du_salon_profile0
    tap_action:
      action: call-service
      service: onvif.ptz
      service_data:
        entity_id: camera.camera_onvif_du_salon_profile0
        tilt: DOWN
        speed: 1
        distance: 0.3
        move_mode: ContinuousMove
    name: Bas
    show_state: false
    icon: mdi:arrow-down
```

-- Configuration de la carte HA positions prédéfinies --

```yaml
type: glance
title: Caméra du salon
entities:
  - entity: camera.camera_onvif_du_salon_profile0
    tap_action:
      action: call-service
      service: onvif.ptz
      service_data:
        entity_id: camera.camera_onvif_du_salon_profile0
        preset: Preset1
        speed: 1
        distance: 0.3
        move_mode: GotoPreset
    name: Pos 1
    show_state: false
    icon: mdi:numeric-1-box
  - entity: camera.camera_onvif_du_salon_profile0
    tap_action:
      action: call-service
      service: onvif.ptz
      service_data:
        entity_id: camera.camera_onvif_du_salon_profile0
        preset: Preset2
        speed: 1
        distance: 0.3
        move_mode: GotoPreset
    name: Pos 2
    show_state: false
    icon: mdi:numeric-2-box
  - entity: camera.camera_onvif_du_salon_profile0
    tap_action:
      action: call-service
      service: onvif.ptz
      service_data:
        entity_id: camera.camera_onvif_du_salon_profile0
        preset: Preset3
        speed: 1
        distance: 0.3
        move_mode: GotoPreset
    name: Pos 3
    show_state: false
    icon: mdi:numeric-3-box
```

La caméra Heden dispose d'un serveur Web qui permet d'accéder à la configuration de celle-ci et il est notamment possible de modifier le numéro de port utilisé par le protocole ONVIF.

### Bdd de Home Assistant

Home Assistant utilise par défaut une base de données SQlite.

Le fichier Bdd est enregistré par défaut dans le dossier config de Home Assistant sous le nom :  
home-assistant_v2.db

### MQTT et SSL

Installer le plugin Mosquitto broker _(serveur MQTT)_ sur Home Assistant depuis le menu Paramètres -> Applications.

Créer, toujours sur Home Assistant, un utilisateur MQTT Loup possédant des droits d'administrateur et qui sera dédié à l'exploitation du protocole MQTT.

Créer une clé CA _(Autorité de certification)_ :

```bash
openssl genrsa -des3 -out ca.key 2048
```

Une passphrase (phrase secrète) sera demandée.

Fichier ca.key créé.

Créer ensuite le certificat CA :

```bash
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
```

Retour :

```markdown
PEM pass phrase = Mdp habituel
Ville = Toulouse
Organization Name = CA HomeAss
Organizational Unit Name = Labo CA
Common Name = xxx (nom d'hôte du serveur Mosquitto)
Email Address = user1@bla.com
```

Fichier ca.crt créé.

Créer la clé privée du serveur Mosquitto :

```bash
openssl genrsa -out server.key 2048
```

La passphrase précédente est demandée.

Fichier server.key créé.

Créer le certificat de demande de signature associé à la clé privée du serveur

```bash
openssl req -new -out server.csr -key server.key
```

Retour :

```markdown
PEM pass phrase = Mdp habituel
Ville = Bordeaux
Organization Name = HomeAss
Organizational Unit Name = Utilisateur
Common Name = xxx (nom d'hôte du serveur Mosquitto)
Email Address = user2@bla.com
```

Fichier server.csr créé.

Créer le certificat final signé pour Mosquitto

```bash
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 3650
```

La passphrase est demandée.

Fichiers server.crt et ca.srl créés.

Créer une clé privée client :

A priori, note technique non finie ??

### QNAP

Ne pas oublier l'utilisateur créé sur le Qnap et dédié aux requêtes d'Home Assistant.

Celui-ci est utilisé par le module d'intégration Qnap fourni par Home Assistant.

**Fin.**
