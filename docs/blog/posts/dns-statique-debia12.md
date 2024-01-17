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
\- Positif sur un nom de domaine Internet  
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

Le système fournit bind 9 et unbound soit les 2 serveurs DNS les plus utilisés sous Debian.

Le choix se portera sur bind 9 qui contrairement à unbound peut être utilisé à la fois comme serveur de noms récursif et serveur de noms faisant autorité.

Un serveur DNS récursif recherchera le résultat d'une requête DNS dans son cache ou à défaut interrogera un serveur DNS faisant autorité pour obtenir le résultat.

Un serveur DNS faisant autorité contiendra le résultat, il n'interrogera pas d'autres serveurs et sera l’autorité finale contenant tous les noms d'hôtes et adresses IP d'une zone donnée.

Le service DNS sera statique car les données seront pour l'instant renseignées manuellement.

#### _- Installation de bind 9_

Installez le paquet bind9 :

```bash
[srvlan@srvlan:~$] sudo apt install bind9
```

et vérifiez le démarrage de celui-ci :

```bash
[srvlan@srvlan:~$] systemctl status bind9
```

Retour :

```markdown
● named.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/named.ser...
     Active: active (running) since Thu 2024-01-...
       Docs: man:named(8)
   Main PID: 3234 (named)
     Status: "running"
      Tasks: 4 (limit: 1077)
     Memory: 33.0M
        CPU: 87ms
     CGroup: /system.slice/named.service
        └─3234 /usr/sbin/named -f -u bind
```

Vérifiez son lancement automatique au boot de la VM :

```bash
[srvlan@srvlan:~$] systemctl is-enabled named
```

Retour :

```markdown
enabled
```

Vérifiez la version installée :

```bash
[srvlan@srvlan:~$] sudo named -v
```

Retour :

```markdown
BIND 9.18.19...-Debian (Extended Support V...) <id:>
```

Vérifiez l'utilisation des ports TCP/UDP 53 et 953 :

```bash
[srvlan@srvlan:~$] sudo ss -tulpn | grep named
```

Retour partiel :

```markdown
udp UNC... 0 0        192.168.3.1:53   ..."named"...      
udp UNC... 0 0        192.168.2.2:53   ..."named"...      
udp UNC... 0 0          127.0.0.1:53   ..."named"...      
udp UNC... 0 0              [::1]:53   ..."named"...      
udp UNC... 0 0   [fe80::...enp0s3:53   ..."named"...     
udp UNC... 0 0   [fe80::...enp0s8:53   ..."named"...      
tcp LIS... 0 10       192.168.3.1:53   ..."named"...      
tcp LIS... 0 10       192.168.2.2:53   ..."named"...      
tcp LIS... 0 10         127.0.0.1:53   ..."named"...      
tcp LIS... 0 5         127.0.0.1:953   ..."named"...      
tcp LIS... 0 10             [::1]:53   ..."named"...      
tcp LIS... 0 10  [fe80::...enp0s3:53   ..."named"...      
tcp LIS... 0 10  [fe80::...enp0s8:53   ..."named"...      
tcp LIS... 0 5             [::1]:953   ..."named"...
```


