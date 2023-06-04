---
title: "DHCP / Debian 10"
date: "2022-04-23"
categories: 
  - "archives"
---

## Mémento 7.2 - ISC DHCP Server

Le DHCP sera activé sur la VM srvlan pour la zone LAN.

### 1 - Préambule

Le choix du serveur se portera de nouveau sur le paquet isc-dhcp-server issu du même organisme que bind soit l'Internet Software Consortium _(ISC)_.

### 2 - Installation et configuration du service DHCP

#### _2.1 - Installation du serveur isc-dhcp-server_

Installez le paquet isc-dhcp-server :

\[srvlan@srvlan:~$\] sudo apt install isc-dhcp-server

Retour :

```
...
Generating /etc/default/isc-dhcp-server...
Job for isc-dhcp-server.service failed because ...
See "systemctl status isc-dhcp-server.service" ...
invoke-rc.d: initscript isc-dhcp-server, ... failed.
● isc-dhcp-server.service - LSB: DHCP server
   Loaded: loaded (/etc/init.d/isc-dhcp-server; ...
   Active: failed (Result: exit-code) since Fri ...
     Docs: man:systemd-sysv-generator(8)
  Process: ...isc-dhcp-server start (.../FAILURE)

oct. 25 19:04:51 srvlan dhcpd[1183]: bugs on ...
oct. 25 19:04:51 srvlan dhcpd[1183]: before submi...
oct. 25 19:04:51 srvlan dhcpd[1183]: process and ...
oct. 25 19:04:51 srvlan dhcpd[1183]: 
oct. 25 19:04:51 srvlan dhcpd[1183]: exiting.
oct. 25 19:04:53 srvlan isc-dhcp-server[...
oct. 25 19:04:53 srvlan isc-dhcp-dhcp-... failed!
oct. 25 19:04:53 srvlan systemd[1]: isc-dhcp-...
oct. 25 19:04:53 srvlan systemd[1]: ... Failed...
oct. 25 19:04:53 srvlan systemd[1]: Failed to ...
Traitement ... (« triggers ») pour man-db (2...) ...
Traitement ... (« triggers ») pour libc-bin (2...
Traitement ... (« triggers ») pour systemd (241...
```

Constat : Le démarrage automatique du service plante, une configuration post-installation est nécessaire avant de pouvoir redémarrer sans échec.

#### _2.2 - Configuration_

3 fichiers de configuration ont été créés :  
\- isc-dhcp-server dans le dossier /etc/default/.  
\- dhcpd.conf et dhcpd6.conf dans le dossier /etc/dhcp/.

Editez en premier le fichier isc-dhcp-server :

\[srvlant@srvlan:~$\] sudo nano /etc/default/isc-dhcp-server

et modifiez la ligne INTERFACESv4="" comme suit :

INTERFACESv4="enp0s8"

La carte enp0s8, située côté LAN, sera ainsi surveillée par le serveur DHCP.

Editez maintenant le fichier dhcpd.conf :

\[srvlant@srvlan:~$\] sudo nano /etc/dhcp/dhcpd.conf

et modifiez le comme suit :

\### dhcpd.conf

## Déclarations optionnelles :
# Domaine que les clients doivent utiliser pour résoudre
# les noms d'hôte via le DNS.
option domain-name "intra.loupipfire.fr";

# Liste des serveurs DNS disponibles pour les clients.
option domain-name-servers srvlan.intra.loupipfire.fr;

## Bail :
# Durée du bail par défaut si le client ne spécifie rien.
# Ex : 1 jour = 86400 secondes. 
default-lease-time 86400;

# Durée maxi du bail que peut demander le client
# Ex : 7 jours = 604800 secondes.
max-lease-time 604800;

## Caractéristiques du serveur DHCP :
# Pas de mise à jour dynamique du service DNS.
ddns-update-style none;

# Serveur DHCP faisant autorité pour le sous-réseau local.
authoritative;

# Envoi des logs sur un fichier autre que
# le fichier /var/log/syslog.
log-facility local7;

## Déclaration basique du sous-réseau local à traiter :
# Sous-réseau et masque de sous-réseau.
subnet 192.168.3.0 netmask 255.255.255.0 {

# Etendue de la plage DHCP.
range 192.168.3.30 192.168.3.50;

# Passerelle par défaut.
option routers srvlan.intra.loupipfire.fr;
}

Redémarrez le service DHCP et vérifiez son statut :

\[srvlan@srvlan:~$\] sudo systemctl restart isc-dhcp-server 
\[srvlan@srvlan:~$\] sudo systemctl status isc-dhcp-server 

Retour :

```
● isc-dhcp-server.service - LSB: DHCP server
   Loaded: loaded (/etc/init.d/isc-dhcp-server; ...
   Active: active (running) since Fri 2019-10-25 ...
     Docs: man:systemd-sysv-generator(8)
  Process: 1373 ExecStart=/etc/init.d/isc...SUCCESS)
    Tasks: 1 (limit: 679)
   Memory: 5.5M
   CGroup: /system.slice/isc-dhcp-server.service
           └─1385 /usr/sbin/dhcpd -4 -q ... enp0s8

oct. 25 ... srvlan dhcpd[1382]: All rights reserved.
oct. 25 ... srvlan dhcpd[1382]: For info, ...
oct. 25 ... srvlan dhcpd[1385]: Internet Systems ...
oct. 25 ... srvlan dhcpd[1385]: Copyright 2004-...
oct. 25 ... srvlan dhcpd[1385]: All rights reserved.
oct. 25 ... srvlan dhcpd[1385]: For info, ...
oct. 25 ... srvlan dhcpd[1385]: Wrote 0 leases ...
oct. 25 ... srvlan dhcpd[1385]: Server starting ...
oct. 25 ... srvlan isc-dhcp-server[1373]: Start...
oct. 25 ... srvlan systemd[1]: Started LSB: DHCP ...
```

Le statut montre cette fois-ci un démarrage sans erreur.

### 3 - Gestion des logs du service DHCP

#### _3.1 - Création d'un fichier log_

Créez un fichier isc-dhcp-server.log :

\[srvlan@srvlan:~$\] sudo mkdir /var/log/dhcp
\[srvlan@srvlan:~$\] cd /var/log/dhcp
\[srvlan@srvlan:~$\] sudo touch isc-dhcp-server.log

puis modifiez ses propriétés comme suit :

\[srvlan@srvlan:~$\] sudo chown root:adm isc-dhcp-server.log

Propriétaire = root, Groupe = adm

ainsi que ses droits d'accès comme suit :

\[srvlan@srvlan:~$\] sudo chmod 640 isc-dhcp-server.log  

6 = Lecture/Ecriture pour le propriétaire/utilisateur root  
4 = Lecture pour les utilisateurs du groupe adm  
0 = Accès interdits pour les autres utilisateurs

#### _3.2 - Traitement du fichier avec rsyslog_

Vérifiez le statut du démon rsyslogd :

\[srvlan@srvlan:~$\] sudo systemctl status rsyslog 

Les instructions de configuration du démon se trouvent dans le fichier /etc/rsyslog.conf.

L'une d'entre elles spécifie de traiter tous les fichiers inclus dans le dossier /etc/rsyslog.d/.

Créez alors un fichier de nom isc-dhcp-server-log.conf :

\[srvlan@srvlan:~$\] cd /etc/rsyslog.d
\[srvlan@srvlan:~$\] sudo nano isc-dhcp-server-log.conf

et entrez les 3 lignes suivantes :

\# Envoi des logs vers le fichier isc-dhcp-server.log, sachant le
# paramètre "log-facility local7" activé dans /etc/dhcpd.conf.
local7.\* /var/log/dhcp/isc-dhcp-server.log

Puis éditez /etc/rsyslog.conf et modifiez la ligne :

\*_.\*_;auth,authpriv.none        -/var/log/syslog 

comme suit :

\*_.\*_;auth,authpriv.none;local7.none        -/var/log/syslog 

Les logs DHCP iront ainsi vers le fichier isc-dhcp-server.log et non vers le fichier syslog.

Relancez le démon rsyslogd et vérifiez son statut :

\[srvlan@srvlan:~$\] sudo systemctl restart rsyslog
\[srvlan@srvlan:~$\] sudo systemctl status rsyslog 

### 4 - Modification du DNS statique

Ref : [Mémento 7.1](/dns-statique-debian10/)

Ouvrez le fichier DNS de zone directe intra.loupipfire.fr :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo nano db.intra.loupipfire.fr.directe   

et supprimez ou commentez _(;)_ les 2 lignes suivantes :

debian10-vm1 IN A 192.168.3.2
debian10-vm2 IN A 192.168.3.4

Ouvrez le fichier DNS de zone inverse intra.loupipfire.fr :

\[srvlan@srvlan:~$\] sudo nano db.intra.loupipfire.fr.inverse   

et supprimez ou commentez _(;)_ les 2 lignes suivantes :

2 IN PTR debian10-vm1.intra.loupipfire.fr. 
4 IN PTR debian10-vm2.intra.loupipfire.fr. 

Relancez le service DNS :

\[srvlan@srvlan:~$\] sudo systemctl restart bind9

_Nota : Ne modifiez pas les lignes des hôtes srvlan/ovs._

### 5 - Envoi automatique des configurations IP

Ceci impose de modifier l'adressage IP initial des clients du réseau local.

Au préalable, ouvrez sur srvlan un second terminal et connectez-vous sur celui-ci en tant qu'utilisateur root _(Cde : su root)_.

Entrez maintenant la demande de traçage suivante :

\[root@srvlan:~#\] tail -f /var/log/dhcp/isc-dhcp-server.log

Cela permettra d'observer en temps réel les logs DHCP.

#### _5.1 - Client debian10-vm1_

Référez-vous au § 1.8 du [Mémento 4.1](/clients-debian-10-installation-virtualbox/#18_-_Configuration_du_reseau_et_adressage_IP_fixe) pour modifier l'adresse IP.

Modifiez la Méthode de l'onglet Paramètres IPv4 sur Automatique (DHCP).

Supprimez ensuite l'adresse IP statique et enregistrez.

Redémarrez la VM :

\[client-linux@debian10-vm1:~$\] sudo reboot

et contrôlez l'affectation d'une nouvelle adresse IP :

\[client-linux@debian10-vm1 :~$ \] ip address

qui doit faire partie de la plage DHCP du serveur DHCP.

La VM debian10-vm1, premier client DHCP servi, a reçu la première adresse IP de cette plage soit 192.168.3.30.

Démarrez Firefox et vérifiez le bon accès à Internet.

\- VM srvlan - 
Vérifiez que le terminal de traçage contient ces lignes :

```
dhcpd ... DHCPDISCOVER from ...5e:7b via enp0s8
dhcpd ... DHCPOFFER on 192.168.3.30 to ...5e:7b ...
dhcpd ... DHCPREQUEST for 192.168.3.30 ...5e:7b ...
dhcpd ... Wrote 1 leases to leases file.
dhcpd ... DHCPACK on ...3.30 ... (debian10-vm1) ...
```

et que le fichier des baux /var/lib/dhcp/dhcpd.leases contient celles-ci:

```
# authoring-byte-order entry is ..., DO NOT DELETE
authoring-byte-order little-endian;

lease 192.168.3.30 {
  starts 6 2019/10/26 11:35:27;   (bail de 1 jour)
  ends 0 2019/10/27 11:35:27;
  cltt 6 2019/10/26 11:35:27;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:9d:5e:7b;
  client-hostname "debian10-vm1";
}
```

#### _5.2 - Client debian10-vm2_

Procédez de la même manière que pour debian10-vm1. 
Cette fois, l'IP attribuée devrait être 192.168.3.31.

### 6 - Bilan

Le serveur DHCP remplit son rôle, en revanche, la résolution DNS locale sur les VM debian10-vm\* est perdue suite aux modifications du DNS statique.

Il est, pour corriger cela et disposer à l'avenir d'une modification automatique des fichiers DNS, nécessaire de mettre en place un DNS dynamique.

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Voilà, fini pour aujourd'hui.  
Le mémento 7.3 vous attend à  
présent pour modifier le DNS statique  
en DNS dynamique.

[Mémento 7.3](/dns-dynamique-via-dhcp/)
