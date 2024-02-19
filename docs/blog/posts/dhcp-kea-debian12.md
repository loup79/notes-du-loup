---
title: "DHCP Kea - VBox/Deb12"
summary: Le DHCP sera activé sur la VM srvlan pour la zone LAN.
authors: 
  - G.Leloup
date: 2024-02-19
categories: 
  - 7 - DNS - DHCP - DNS dynamique
---

<figure markdown>
  ![Synoptique - DHCP Kea sous Debian 12 - Mémento 7.2](../images/2024/02/dhcp-kea-memento7.2-deb12.webp){ width="430" }
</figure>

## Mémento 7.2 - Kea DHCP Server

DHCP = Dynamic Host Configuration Protocol

Le DHCP sera activé sur la VM srvlan pour la zone LAN.

### Préambule

Il est aujourd'hui préconisé d'utiliser Kea DHCP à la place d'ISC DHCP qui n'est plus maintenu.

#### _Rôle du serveur DHCP_

Il permet à un hôte qui se connecte sur un réseau local d'obtenir automatiquement sa configuration IP, ceci pour une durée déterminée appelée bail DHCP.

But, faciliter l'affectation des adresses IP sur un réseau.

Les hôtes enregistrés via un serveur DHCP peuvent aussi être ajoutés dynamiquement à un serveur DNS.

DHCP représente donc une suite logique au DNS dans la construction du réseau local virtuel.

#### _Fonctionnement du protocole_

Le DHCP repose sur des requêtes UDP émises par les clients et traitées par le serveur.

Exemple d'un échange de requêtes client/serveur :

<!-- more -->

* DHCPDISCOVER - Le client cherche un serveur  
* DHCPOFFER - le 1er trouvé soumet une IP  
* DHCPREQUEST - Le client traite et valide l'IP  
* DHCPACK - Le serveur confirme la configuration  

Les requêtes UDP circulent sur les ports 67 et 68.

Le client au final reçoit une confirmation de son IP temporaire (bail) ainsi que souvent en complément l'IP de sa passerelle ainsi que les IP de ses serveurs DNS.

!!! note "Nota"
    Si pas de serveur DHCP, le client s'attribue une adresse IP dans la plage 169.254.0.0/16.

Le serveur DHCP Kea utilisé ci-dessous est issu de l'organisme déjà créateur du serveur DNS BIND 9 soit l'Internet Software Consortium _(ISC)_.

### Installation et configuration du service

#### _Installation du service Kea_

Installez ce paquet pour gérer la zone LAN en IPv4 :

```bash
[srvlan@srvlan:~$] sudo apt install kea-dhcp4-server 
```

Les dépendances suivantes ont été ajoutées :

* kea-common
* liblog4cplus
* libmariadb3
* libpq5
* mariadb-common
* mysql-common

Vérifiez le statut du service :

```bash
[srvlan@srvlan:~$] sudo systemctl status kea-dhcp4-server
```

Retour normal :

```markdown
● kea-dhcp4-server.service - Kea IPv4 DHCP daemon
     Loaded: ...service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-...
       Docs: man:kea-dhcp4(8)
   Main PID: 2412 (kea-dhcp4)
      Tasks: 5 (limit: 1077)
     Memory: 4.6M
        CPU: 53ms
     CGroup: /system.slice/kea-dhcp4-server.service
             └─2412 /usr/sbin/kea-dhcp4 -c /etc...

...
févr.. srvlan kea-dhcp4[...]: INFO  DHCP4_STARTED
```

Le service est démarré et activé par défaut _(enabled)_.

