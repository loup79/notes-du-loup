---
title: Debian - Docker + Portainer
description: Installation de Docker et Portainer.
authors: 
  - G.Leloup
date: 2026-06-28
categories: 
  - Debian
---

Opération effectuée sous Debian 13 _(trixie)_.

## Docker

Mettre à jour Debian :

```bash
sudo apt update && sudo apt upgrade -y
```

Installer les dépendances :

```bash
sudo apt install -y ca-certificates curl gnupg
```

Installer Docker :

```bash
sudo apt install -y docker.io docker-cli containerd docker-buildx docker-compose
```

Vérifier l'installation :

```bash
sudo docker run hello-world
```

Retour normal :

```markdown hl_lines="7 8"
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
Digest: sha256:96498ffd522e70807ab6384a5c0485a79b9c7c08ca79ba08623edcad1054e62d
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

<!-- more -->

(Optionnel) Ajouter l'utilisateur courant au groupe docker pour éviter d'utiliser sudo avec Docker :

```bash
sudo usermod -aG docker $USER
newgrp docker  # Recharge les groupes sans redémarrer
```

Vérifier les versions de docker et docker-compose :

```bash
docker version
docker compose version
```

Retour normal :

```markdown hl_lines="2 12 29"
Client:
 Version:           26.1.5+dfsg1
 API version:       1.45
 Go version:        go1.24.4
 Git commit:        a72d7cd
 Built:             Wed Jul 30 19:37:00 2025
 OS/Arch:           linux/amd64
 Context:           default

Server:
 Engine:
  Version:          26.1.5+dfsg1
  API version:      1.45 (minimum version 1.24)
  Go version:       go1.24.4
  Git commit:       411e817
  Built:            Wed Jul 30 19:37:00 2025
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.24~ds1
  GitCommit:        1.7.24~ds1-6+b4
 runc:
  Version:          1.1.15+ds1
  GitCommit:        1.1.15+ds1-2+b4
 docker-init:
  Version:          0.19.0
  GitCommit:        

Docker Compose version 2.26.1-4
```

## Portainer

Créer un volume pour Portainer :

```bash
docker volume create portainer_data
```

Créer et lancer le conteneur Portainer :

```bash
docker run -d \
  -p 9443:9443 \
  -p 8000:8000 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Vérifier le résultat :

```bash
docker ps
```

Ouvrir ensuite un navigateur et allez sur :  
`https://localhost:9443`

## MAJ du conteneur Portainer

Mettre à jour l'image de Portainer :

```bash
docker pull portainer/portainer-ce:latest
```

Arrêter et supprimer le conteneur :

```bash
docker stop portainer
docker rm portainer
```

!!! Nota "Nota"
    Ne pas supprimer le volume portainer_data qui contient les configurations et données.

Relancer Portainer avec la dernière version :

```bash
docker run -d \
  -p 9443:9443 \
  -p 8000:8000 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Ensuite se connecter et supprimer l'ancienne image de Portainer.

**Fin.**
