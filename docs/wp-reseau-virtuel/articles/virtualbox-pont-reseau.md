---
title: "VirtualBox - Accès réseau par pont"
date: "2020-08-07"
categories: 
  - "hyperviseur-virtualbox"
---

## Pont réseau entre VM et PC hôte

<table><tbody><tr><td><a href="https://www.virtualbox.org/" target="_blank" rel="noreferrer noopener"><img class="wp-image-35" style="width: 120px" src="../../wp-content/uploads/2019/02/logo-virtualbox.jpg" alt="Logo - VirtualBox" title="https://www.virtualbox.org/"></a></td><td>Configuration d'un pont réseau <em>(bridge)</em></td></tr></tbody></table>

### 1 - Rôle

Le mode d'accès réseau par pont de VirtualBox crée un pont entre la carte réseau d'une VM et celle du PC hôte. La VM peut ainsi appartenir au même réseau local que celui du PC hôte.

Le serveur DHCP utilisé par le PC hôte peut fournir une adresse IP dynamique à la VM mais ce n'est pas obligatoire, l'adresse IP pouvant être paramétrée comme fixe au niveau de la VM.

### 2 - Intérêt

Le pont offre l'avantage de pouvoir réaliser des tests depuis le PC hôte vers le réseau local virtuel comme celui de simuler des requêtes HTTP/FTP issues d'Internet.

### 3 - Installation d'un pont entre l'hôte et srvlan

#### _3.1 - Modification sur srvlan de l'accès réseau_

Arrêtez la VM srvlan.

Depuis VirtualBox, affichez la configuration de la VM :  
\- Section Réseau  
\> Carte 1 > Mode d'accès réseau > Accès par pont  
\> Nom > Sélectionnez la carte réseau active du PC hôte  
\> OK

Redémarrez la VM.

#### 3.2 - Modification sur srvlan de l'adresse IP fixe

Editez le fichier réseau interfaces :

`[root@srvlan:~#\] nano /etc/network/interfaces`

et modifiez les lignes ci-dessous comme suit :

```
auto eth
iface eth0 inet static
address 192.168.x.w          \# IP libre du réseau local PC hôte
netmask 255.255.255.0
network 192.168.x.0           \# Réseau local du PC hôte
broadcast 192.168.x.255
gateway 192.168.x.z          \# IP de la box Internet, Freebox, etc...
```

Relancez le service réseau sur la carte eth0 :

`[root@srvlan:~#\] service network-interface restart INTERFACE=eth0`

#### _3.3 - Test de la liaison entre l'hôte et srvlan_

Effectuez un ping depuis le PC hôte vers srvlan et inversement.

\---------- Fin ----------
