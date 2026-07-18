---
title: Debian - Podman + Hass
description: Installation de Podman et Home Assistant.
authors: 
  - G.Leloup
date: 2024-06-28
categories: 
  - Debian
---

## Podman et Home Assistant

### 1 - Installation

Podman est installé sur le RPI4 Raspirant.

Installation :

```bash
sudo apt install podman  
sudo apt install cockpit-podman
```

Test :

```bash
podman info
```

La Cde *podman info* peut renvoyer l'erreur suivante :

```markdown
unable to write pod event: "write unixgram @00013->/run/systemd/journal/socket: sendmsg: no such file or directory"
```

Pour supprimer cette erreur, créer le fichier */etc/containers/containers.conf* et entrer ce contenu :

```markdown
[engine]
events_logger = "file"
```

Cdes Podman diverses :  

<!-- more -->

podman version  
podman info  
podman ps  
podman system connection list  
podman start *nom-du-conteneur*  
podman stop *nom-du-conteneur*  
podman image -a  
podman inspect *nom-du-conteneur*  
podman system df

Podman stocke les données temporaires dans /var/tmp/.

Voir avec la Cde :

```bash
sudo du -h -S -c /var/tmp/
```

Les conteneurs Podman peuvent être gérés depuis l'interface de l'outil d'administration Cockpit.

### 2 - Création du conteneur Hass

L'exemple ci-dessous concerne le serveur Home Assistant.

Au prélable, créer les dossiers suivants dans un dossier *user* :

```bash
cd /home/user
mkdir -p /home/user/Podman/user-hass/config
mkdir -p /home/user/Podman/user-hass/media
```

Ensuite créer le conteneur *user-hass* comme suit :

```bash
podman run -d \
--name user-hass \
--cap-add=CAP_NET_RAW,CAP_NET_BIND_SERVICE \
--restart=unless-stopped -p 9225:9225 \
-v /etc/localtime:/etc/localtime:ro \
-v /home/user/Podman/user-hass:/config \
-v /home/user/Podman/user-hass:/media \
--pull=always homeassistant/home-assistant:stable
```

Une fois créé, le conteneur est accessible comme suit :

`http://127.0.0.1:9225`

**Fin.**
