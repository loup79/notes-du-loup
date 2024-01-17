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

#### _- Situation actuelle zone LAN_

a) Une IP fixe et un nom d'hôte pour chaque VM/CTN.  
Ex : IP fixe 192.168.3.2 pour l'hôte debian12-vm1

Cde utile pour découvrir les adresses IP :

```bash
[user@hostname:~$] ip address
```

Cdes utiles pour découvrir les noms d'hôtes :

```bash
[user@hostname:~$] cat /etc/hostname
[user@hostname:~$] cat /etc/hosts               # Ligne 127.0.1.1
```

b) Une Box Internet déclarée comme serveur DNS.

Cde utile pour découvrir les serveurs DNS exploités :

```bash
[user@hostname:~$] cat /etc/resolv.conf
```

c) Des pings qui émis depuis le LAN montrent un retour :  
- Positif sur un nom de domaine Internet  
Ex : ping www.google.fr

\- Positif sur leur propre nom d'hôte  
Ex : ping debian12-vm1 depuis la VM debian12-vm1

\- Négatif sur les autres noms d'hôtes de la zone LAN

Exemple de retour :

```markdown
client-linux@debian12-vm1:~$ ping srvlan
ping: srvlan: Nom ou service inconnu
client-linux@debian12-vm1:~$
```

Un serveur DNS local solutionnera le retour négatif.

### Mise en place du DNS statique

