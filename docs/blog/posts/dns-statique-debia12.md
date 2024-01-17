---
title: "DNS statique - VBox/Deb12"
summary: Le DNS statique gérera un sous-domaine de nom intra.loupipfire.fr.
author: G.Leloup
date: 2024-01-17
categories: 
  - 8 - DNS - DHCP - DNS dynamique
---

<figure markdown>
  ![Synoptique - DNS statique sous Debian 12 - Mémento 7.1](../images/2024/01/dns-statique-deb12-memento-7.1.webp){ width="430" }
</figure>

## Mémento 7.1 - DNS avec BIND 9

Le service DNS statique reposera sur bind 9 qui sera installé sur srvlan pour gérer la zone LAN.

### Préambule

Le service gérera un sous-domaine de loupipfire.fr nommé intra.loupipfire.fr. Le nom de ce sous-domaine sera associé aux adresses IP de la zone LAN.

La VM srvlan doit conserver son IP fixe 192.168.3.1.

Commencez par une MAJ des CTN de la zone LAN :

```bash
sudo podman exec -it ctn1 ou 2 bash   # Conteneurs rootfull
# apt update
# apt upgrade
# exit

podman exec -it ctn3 bash             # Conteneur rootless
# apt update
# apt upgrade
# exit 
```

et ensuite celle des VM de la zone LAN :

<!-- more -->

```bash
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade
sudo reboot
```

VM = Machine virtuelle et CTN = Conteneur Podman
