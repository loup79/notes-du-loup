---
title: "DHCP / Debian 11"
date: "2022-02-16"
categories: 
  - "services-dns-dhcp"
---

## Mémento 7.21 - ISC DHCP Server

DHCP = Dynamic Host Configuration Protocol

Le DHCP sera activé sur la VM srvlan pour la zone LAN.

### 1 - Préambule

Référence : [DHCP / Debian 10](/dhcp-isc-dhcp-server-debian10/).

#### _1.1 - Rôle d'un serveur DHCP_

Il permet à un hôte qui se connecte sur un réseau local d'obtenir automatiquement sa configuration IP. Le but étant de simplifier l'administration d'un réseau.

Les hôtes enregistrés via un serveur DHCP peuvent aussi être ajoutés dynamiquement à un serveur DNS.

DHCP représente donc une suite logique au DNS dans la construction du réseau local virtuel.

#### _1.2 - Fonctionnement du protocole DHCP_

Le DHCP repose sur des requêtes UDP émises par les clients et traitées par le serveur.

Exemple d'échange de requêtes client/serveur :

- DHCPDISCOVER - Le client cherche un serveur

- DHCPOFFER - le 1er trouvé soumet une IP

- DHCPREQUEST - Le client traite et valide l'IP

- DHCPACK - Le serveur confirme la configuration

Les requêtes UDP circulent sur les ports 67 et 68.

Le client au final reçoit une confirmation de son IP temporaire _(bail)_ ainsi que souvent en complément l'IP de sa passerelle ainsi que les IP de ses serveurs DNS.

_Nota : Si pas de serveur DHCP, le client s'attribue une adresse IP dans la plage 169.254.0.0/16._

Le serveur DHCP isc-dhcp-server utilisé ci-dessous est issu du même organisme que le serveur DNS bind soit l'Internet Software Consortium _(ISC)_.

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
     Loaded: loaded (/etc/init.d/isc-dhcp-serve...
     Active: failed (Result: exit-code) ... 2022-...
       Docs: man:systemd-sysv-generator(8)
    Process: ...isc-dhcp-server start (.../FAILURE)
        CPU: 142ms

janv. 24 13:50:44 srvlan dhcpd[1563]: before subm...
janv. 24 13:50:44 srvlan dhcpd[1563]: process an...
janv. 24 13:50:44 srvlan dhcpd[1563]: 
janv. 24 13:50:44 srvlan dhcpd[1563]: exiting.
janv. 24 13:50:46 srvlan isc-dhcp-server[...
janv. 24 13:50:46 srvlan isc-dhcp-...  failed!
janv. 24 13:50:46 srvlan isc-dhcp-...  failed!
janv. 24 13:50:46 srvlan systemd[1]: .../FAILURE>
janv. 24 13:50:46 srvlan systemd[1]: ... Failed ...
janv. 24 13:50:46 srvlan systemd[1]: Failed to st...
Traitement ... (« triggers ») pour libc-bin (2...
Traitement ... (« triggers ») pour man-db (2...
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

et remplacez tout son contenu par celui-ci :

\### Fichier /etc/dhcpd/dhcpd.conf

## Déclarations optionnelles :
# Domaine que les clients doivent utiliser pour résoudre
# les noms d'hôte via le DNS.
option domain-name "intra.loupipfire.fr";

# Liste des serveurs DNS disponibles pour les clients.
option domain-name-servers srvlan.intra.loupipfire.fr;

## Bail :
# Durée du bail (lease time) si le client ne spécifie rien.
# Ex : 1 jour = 86400 secondes. 
default-lease-time 86400;

# Durée maxi du bail que peut demander le client.
# Ex : 7 jours = 604800 secondes.
max-lease-time 604800;

## Caractéristiques du serveur DHCP :
# Pas de mise à jour dynamique du service DNS.
ddns-update-style none;

# Serveur DHCP faisant autorité pour le sous-réseau local.
authoritative;

# Envoi des logs DHCP sur un fichier autre que
# le fichier /var/log/syslog.
log-facility local7;

## Déclaration du sous-réseau local à traiter :
# Sous-réseau et masque de sous-réseau.
subnet 192.168.3.0 netmask 255.255.255.0 {

# Etendue de la plage d'adresse IP.
range 192.168.3.30 192.168.3.50;

# Passerelle par défaut.
option routers srvlan.intra.loupipfire.fr;
}

# Réservation d'une IP fixe pour le conteneur LXC ctn1 :
# Cde "ip address" sur ctn1 pour relever l'adresse MAC de eth0.
host ctn1 {
       hardware ethernet 4a:49:43:49:79:bf;
       fixed-address 192.168.3.20;
}

\- A la moitié du bail, le client demandera la prolongation ou renouvellement de celui-ci.

\- L'IP fixe réservée pour ctn1 se situe en dehors de la plage des IP dédiée aux autres clients.

\- Attention, l'adresse MAC de ctn1 actuellement dynamique devra être déclarée fixe ci-dessous.

Redémarrez le service DHCP et vérifiez son statut :

\[srvlan@srvlan:~$\] sudo systemctl restart isc-dhcp-server 
\[srvlan@srvlan:~$\] sudo systemctl status isc-dhcp-server 

Retour du statut :

```
● isc-dhcp-server.service - LSB: DHCP server
     Loaded: loaded (/etc/init.d/isc-dhcp-server...
     Active: active (running) since Mon 2022-01-...
       Docs: man:systemd-sysv-generator(8)
    Process: 2185 ExecStart=/etc/init.d/... SUCCESS)
      Tasks: 4 (limit: 1116)
     Memory: 5.5M
        CPU: 183ms
     CGroup: /system.slice/isc-dhcp-server.service
        └─2201 /usr/sbin/dhcpd -4 -q ... enp0s8

janv. 24 ... srvlan dhcpd[2201]: Wrote 0 deleted ...
janv. 24 ... srvlan dhcpd[2201]: Wrote 0 new dyna...
janv. 24 ... srvlan dhcpd[2201]: Wrote 0 leases t...
janv. 24 ... srvlan dhcpd[2201]: Server starting ...
janv. 24 ... srvlan isc-dhcp-server[2185]: Start...
janv. 24 ... srvlan systemd[1]: Started LSB: DHCP ..
...
```

Celui-ci ne montre pas d'erreur et le service sera lancé automatiquement au boot de srvlan.

Vérifiez le port utilisé par le service DHCP :

\[srvlan@srvlan:~$\] sudo ss -anup | grep dhcp

Retour :

```
UN... 0  0  0.0.0.0:67  ...:*  users:(("dhcpd",...))
```

### 3 - Gestion des logs du service DHCP

#### _3.1 - Création d'un fichier log_

Créez un fichier de nom isc-dhcp-server.log :

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

Relancez le démon rsyslogd pour traiter la modification :

\[srvlan@srvlan:~$\] sudo systemctl restart rsyslog
\[srvlan@srvlan:~$\] sudo systemctl status rsyslog 

### 4 - Modification de la configuration DNS statique

Ref : [Mémento 7.11](/dns-statique-debian11/)

Ouvrez le fichier DNS de zone directe intra.loupipfire.fr :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo nano db.intra.loupipfire.fr.directe   

et supprimez ou commentez _(;)_ les 4 lignes suivantes :

debian11-vm1 IN A 192.168.3.2
debian11-vm2 IN A 192.168.3.4
ctn1 IN A 192.168.3.6
ctn2 IN A 192.168.3.8

Ouvrez le fichier DNS de zone inverse intra.loupipfire.fr :

\[srvlan@srvlan:~$\] sudo nano db.intra.loupipfire.fr.inverse   

et supprimez ou commentez _(;)_ les 4 lignes suivantes :

2 IN PTR debian11-vm1.intra.loupipfire.fr. 
4 IN PTR debian11-vm2.intra.loupipfire.fr.
6 IN PTR ctn1.intra.loupipfire.fr.
8 IN PTR ctn2.intra.loupipfire.fr.

Relancez le service DNS :

\[srvlan@srvlan:~$\] sudo systemctl restart bind9

_Nota : Ne modifiez pas les lignes des hôtes srvlan/ovs._

### 5 - Tests de bon fonctionnement du DHCP

Il faut, pour terminer la configuration DHCP, modifier les paramètres réseau des VM et conteneurs LXC.

Au préalable, ouvrez sur srvlan un second terminal et connectez-vous sur celui-ci en tant qu'utilisateur root _(Cde : su root)_.

Entrez maintenant la demande de traçage suivante :

\[root@srvlan:~#\] tail -f /var/log/dhcp/isc-dhcp-server.log

Cela permettra d'observer en temps réel les logs DHCP.

#### _5.1 - VM debian11-vm1 et 2_

Référez-vous au [Mémento 4.11](/virtualbox-clients-debian11-creation/#16_-_Configuration_du_reseau_et_adressage_IP_fixe) pour modifier les paramètres réseau.

\- VM debian11-vm1

Ajustez la Méthode de l'onglet Paramètres IPv4 sur Automatique (DHCP).

Supprimez ensuite l'adresse IP statique et enregistrez.

Redémarrez la VM Debian :

\[client-linux@debian11-vm1:~$\] sudo reboot

et contrôlez l'affectation d'une nouvelle adresse IP :

\[client-linux@debian11-vm1 :~$ \] ip address

qui doit faire partie de la plage DHCP du serveur DHCP.

Démarrez Firefox et vérifiez le bon accès à Internet.

\- VM srvlan  
Vérifiez que le terminal de traçage contient ces lignes :

```
dhcpd ... DHCPDISCOVER from ...:86:70 via enp0s8
dhcpd ... DHCPOFFER on 192.168.3.31 to ...:86:70 ...
dhcpd ... DHCPREQUEST for 192.168.3.31 ...:86:70 ...
dhcpd ... Wrote 0 deleted host decls to leases file.
dhcpd ... Wrote 0 new dynamic host decls to leas...
dhcpd ... Wrote 2 leases to leases file.
dhcpd ... DHCPACK on ...3.31 ... (debian11-vm1) ...
```

et que le fichier des baux /var/lib/dhcp/dhcpd.leases contient celles-ci :

```
lease 192.168.3.31 {
  starts 3 2022/01/26 12:18:37;    (bail de 1 jour)
  ends 4 2022/01/27 12:18:37;
  cltt 3 2022/01/26 12:18:37;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet ...:86:70;
  uid "\001\010\000'\210\206p";
  client-hostname "debian11-vm1";
}
```

_\- VM debian11-vm2_

Procédez de la même manière que pour debian11-vm1.

#### _5.2 - Conteneurs LXC ctn1 et 2_

\- Conteneur ctn1 sur la VM ovs

Editez la configuration réseau du conteneur :

\[switch@ovs:~$\] sudo nano /var/lib/lxc/ctn1/config

Supprimez ou commentez les 2 lignes ci-dessous :

lxc.net.0.ipv4.address = 192.168.3.6/24
lxc.net.0.ipv4.gateway = 192.168.3.15

et ajoutez une ligne contenant cette adresse MAC fixe :

lxc.net.0.hwaddr = 4a:49:43:49:79:bf    \# Adresse MAC fixe

_Nota : L'adresse MAC doit être celle entrée ci-dessus dans le fichier /etc/dhcpd.conf de srvlan._

Pour la suite, ctn1 est déjà déclaré client DHCP dans son fichier /etc/network/interfaces.

**Remarque** : Si ctn1 a été créé avec Debian 11.3 ou +, le DHCP ne fonctionnera que si vous réactivez le service systemd-networkd désactivé dans le mémento 5.12.

Pour réactiver le service, procédez ainsi :

\[switch@ovs:~$\] sudo lxc-attach -n ctn1

\[root@ctn1:/#\] systemctl start systemd-networkd
\[root@ctn1:/#\] systemctl enable systemd-networkd
\[root@ctn1:/#\] exit

\[switch@ovs:~$\] 

\- Conteneur ctn2 sur la VM ovs

Au préalable, connectez-vous sur celui-ci :

\[switch@ovs:~$\] su - switchctn2
\[switchctn2@ovs:~$\] lxc-unpriv-attach -n ctn2

et relevez son adresse MAC dynamique actuelle :

\[root@ctn2:/#\] ip address
\[root@ctn2:/#\] exit

![Capture : DHCP : Adresse MAC du conteneur ctn2](/wp-content/uploads/2022/01/lxc-ctn2-adresse-mac.webp)

DHCP : Adresse MAC du conteneur ctn2

Editez ensuite sa configuration réseau :

\[switchctn2@ovs:~$\] nano .local/share/lxc/ctn2/config

Supprimez ou commentez les 2 lignes ci-dessous :

lxc.net.0.ipv4.address = 192.168.3.8/24
lxc.net.0.ipv4.gateway = 192.168.3.15

et ajoutez une ligne contenant l'adresse MAC relevée :

lxc.net.0.hwaddr = 6e:91:f3:dc:32:f0    \# Adresse MAC fixe

Pour la suite, ctn2 est également déclaré client DHCP.

**Remarque** : Si ctn2 a été créé avec Debian 11.3 ou +, le DHCP ne fonctionnera que si vous réactivez le service systemd-networkd désactivé dans le mémento 5.12.

Pour réactiver le service, procédez ainsi :

\[switchctn2@ovs:~$\] lxc-unpriv-attach -n ctn2

\[root@ctn2:/#\] systemctl start systemd-networkd
\[root@ctn2:/#\] systemctl enable systemd-networkd
\[root@ctn2:/#\] exit

Redémarrez enfin la VM ovs :

\[switchctn2@ovs:~$\] exit
\[switch@ovs:~$\] sudo reboot

et contrôlez que l'IP affectée automatiquement :  
\- est fixe pour ctn1 : 192.168.3.20. 
\- est dynamique pour ctn2 : IP dans la plage DHCP.

### 6 - Bilan

Le DHCP remplit son rôle mais la résolution DNS locale sur les VM debian11-vm\* et les conteneurs ctn\* est perdue suite aux modifications du DNS statique.

Il est, pour corriger cela et disposer à l'avenir d'une modification automatique des fichiers DNS, nécessaire de mettre en place un DNS dynamique.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Nouvelle étape franchie.  
Le mémento 7.31 vous attend à  
présent pour modifier le DNS statique  
en DNS dynamique.

[Mémento 7.31](https://familleleloup.no-ip.org/dns-dynamique-debian11/)
