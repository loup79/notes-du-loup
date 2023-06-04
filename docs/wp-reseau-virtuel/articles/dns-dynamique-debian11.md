---
title: "DNS dynamique / Debian 11"
date: "2022-02-15"
categories: 
  - "services-dns-dhcp"
---

## Mémento 7.31 - Maj du DNS via DHCP

Migration du DNS statique installé sur la VM srvlan en DNS dynamique via le service DHCP.

### 1 - Préambule

Référence : [DNS dynamique / Debian 10](/dns-dynamique-debian10/).

Toutes les VM doivent être démarrées sauf srvdmz.

#### _1.1 - Rôle du service DNS dynamique via DHCP_

Modifier en temps réel les fichiers de zones DNS via le service DHCP en utilisant les démons named de bind9 et dhcpd de isc-dhcp-server.

### 2 - Configuration du DNS dynamique

#### _2.1 - Configuration actuelle_

Vous avez, lors de la configuration DHCP _([mémento 7.21](/dhcp-isc-dhcp-server-debian11/))_, supprimé volontairement les enregistrements des VM debian11-vm\* et conteneurs LXC ctn\* dans les fichiers de zone DNS directe et inverse.

La résolution DNS interne est donc cassée :

\[srvlan@srvlan:~$\] ping debian11-vm1

Retour :

```
ping: debian11-vm1: Échec temporaire dans la résolution du nom
```

\[srvlan@srvlan:~$\] ping ctn2.intra.loupipfire.fr 

Retour :

```
ping: ctn2.intra.loupipfire.fr: Nom ou service inconnu
```

Seule la résolution DNS externe fonctionne :

\[srvlan@srvlan:~$\] ping lemonde.fr

Retour :

```
PING lemonde.fr (151.101.2.137) 56(84) bytes of d...
64 bytes from 151.101.2.137 (151.101.2.137): icmp...
64 bytes from 151.101.2.137 (151.101.2.137): icmp...
--- lemonde.fr ping statistics ---
2 packets transmitted, 2 received, 0% packet loss...
rtt min/avg/max/mdev = 15.730/16.145/16.560/0.415 ms
```

#### _2.2 - Création d'une clé secrète TSIG_

TSIG _(Transaction Signature)_ est un mécanisme permettant de sécuriser la communication avec un serveur DNS, ceci à base d'une clé secrète partagée.

Il peut être utilisé pour authentifier la source des requêtes et réponses DNS échangées lors d'une MAJ dynamique des fichiers de zones DNS via DHCP.

TSIG requiet, pour fonctionner correctement sur un réseau, une bonne synchronisation du temps _(voir [addendum NTP](/ntp-ipfire-debian11/) pour les plus curieux)_.

Dans le cas présent, vous pouvez laisser VirtualBox gérer par défaut cette synchronisation du temps pour l'ensemble du réseau virtuel, à vous de voir.

Ceci dit, vous allez commencer par générer la clé secrète partagée de TSIG.

Entrez pour cela les Cdes suivantes :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] su root

\[root@srvlan:~$\] sudo tsig-keygen -a HMAC-SHA256 tsig.key  > tsig.key 

\[root@srvlan:~$\] chown bind:bind tsig.key
\[root@srvlan:~$\] chmod 640 tsig.key
\[root@srvlan:~$\] exit

La Cde tsig-keygen est fournie avec le paquet bind9.

Vérifiez ensuite le contenu du fichier tsig.key créé :

\[srvlan@srvlan:~$\] sudo cat tsig.key

Retour :

```
key "tsig.key" {
algorithm hmac-sha256;
secret "z1iGvZ6fqgHtwlQVsltfMqvtBB24rxk2VpZ...";
};
```

L'élément secret contient la valeur de la clé TSIG.

#### _2.3 - Configuration côté serveur DNS_

Editez le fichier named.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf 

et ajoutez les 3 lignes suivantes en fin de celui-ci :

\# Déclaration de la clé TSIG qui sera utilisée pour
# la mise à jour dynamique du DNS
include "/etc/bind/tsig.key"; 

Pour éviter de rencontrer par la suite des problèmes de permission d'écriture, déplacez les fichiers actuels de zone DNS vers le dossier /var/lib/bind/ comme suit :

\[srvlan@srvlan:~$\] cd /var/lib/bind
\[srvlan@srvlan:~$\] sudo mv /etc/bind/\*.directe /var/lib/bind/
\[srvlan@srvlan:~$\] sudo mv /etc/bind/\*.inverse /var/lib/bind/

Sous Debian, ce dossier est destiné aux fichiers de zone et journaux mis à jour dynamiquement.

Cette configuration tient compte du profil de sécurité apparmor associé à l'application bind et paramétré dans le fichier /etc/apparmor.d/usr.sbin.named.

Editez ensuite le fichier named.conf.local :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf.local

puis modifiez la zone de recherche directe comme suit :

file "/var/lib/bind/db.intra.loupipfire.fr.directe";
allow-update { key "tsig.key"; };

et la zone de recherche inverse comme ci-dessous :

file "/var/lib/bind/db.intra.loupipfire.fr.inverse";
allow-update { key "tsig.key"; }; 

Le paramètre allow-update autorise les MAJ DNS si authentifiées avec la clé TSIG tsig.key.

#### _2.4 - Configuration côté serveur DHCP_

Le service DHCP doit utiliser la même clé TSIG que le service DNS.

Pour cela, copiez le fichier tsig.key dans /etc/dhcp/ et gérez ses permissions :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo cp tsig.key /etc/dhcp/

\[srvlan@srvlan:~$\] cd /etc/dhcp
\[srvlan@srvlan:~$\] sudo chown root:root tsig.key
\[srvlan@srvlan:~$\] sudo chmod 640 tsig.key 

Editez ensuite le fichier dhcpd.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/dhcp/dhcpd.conf

et modifiez son contenu comme indiqué ci-dessous :

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

\# Activation de la MAJ dynamique des zone DNS.
ddns-updates on;

# Méthode utilisée = interim pour une MAJ
# sur un serveur DNS local.
ddns-update-style interim;
 
# Ignorer les demandes de MAJ du serveur DNS
# provenant des clients.
ignore client-updates;

# Autoriser la MAJ des enregistrements DNS statiques.
update-static-leases on;

# Autoriser l'attribution d'une adresse IP aux clients inconnus.
allow unknown-clients; 

# Serveur DHCP faisant autorité pour le sous-réseau local.
authoritative;

# Envoi des logs DHCP sur un fichier autre que
# le fichier /var/log/syslog.
log-facility local7;

## Plage d'adresses IP et passerelle IP :
# Déclaration basique du sous-réseau local à traiter
# incluant sous-réseau et masque de sous-réseau.
subnet 192.168.3.0 netmask 255.255.255.0 {

# Etendue de la plage DHCP.
range 192.168.3.30 192.168.3.50;

# Passerelle par défaut.
option routers srvlan.intra.loupipfire.fr;
}

## Réservation d'une IP fixe pour le conteneur LXC ctn1 :
# Cde "ip address" sur ctn1 pour relever l'adresse MAC de eth0.
host ctn1 {
       hardware ethernet 4a:49:43:49:79:bf; # Adresse MAC
       fixed-address 192.168.3.20;
}

\## MAJ dynamique de la zone DNS intra.loupipfire.fr  :
# Informations sur le serveur DNS à mettre à jour.
include "/etc/dhcp/tsig.key";

zone intra.loupipfire.fr. {
primary srvlan.intra.loupipfire.fr;
key tsig.key;
}

zone 3.168.192.in-addr.arpa. {
primary srvlan.intra.loupipfire.fr;
key tsig.key;
} 

Enfin, relancez dans l'ordre les services DNS et DHCP :

\[srvlan@srvlan:~$\] sudo systemctl restart bind9
\[srvlan@srvlan:~$\] sudo systemctl status bind9

\[srvlan@srvlan:~$\] sudo systemctl restart isc-dhcp-server 
\[srvlan@srvlan:~$\] sudo systemctl status isc-dhcp-server

### 3 - Contrôle de bon fonctionnement

#### _3.1 - Client debian11-vm1_

Au préalable, ouvrez 2 terminaux sur le bureau de srvlan.

Connectez-vous ensuite sur ceux-ci en tant qu'utilisateur root _(Cde su root)_.

Entrez sur le terminal 1 cette demande de traçage :

\[root@srvlan:~#\] tail -f /var/log/dhcp/isc-dhcp-server.log

et sur le terminal 2 celle ci-dessous :

\[root@srvlan:~#\] tail -f /var/log/syslog

Redémarrez maintenant la VM debian11-vm1.

Vérifiez ensuite sur le terminal 1 pour le démon dhcpd :  
\- La présence de la mention Added new forward map.  
\- La présence de la mention Added reverse map.

```
Feb 21 14:43:51 srvlan dhcpd: DHCPREQUEST for 192.168.3.32 from 08:00:27... (debian11-vm1) via enp0s8

Feb 21 14:43:51 srvlan dhcpd: DHCPACK on 192.168.3.32 to 08:00:27... (debian11-vm1) via enp0s8

Feb 21 14:43:51 srvlan dhcpd: Added new forward map from debian11-vm1.intra.loupipfire.fr to 192.168.3.32

Feb 21 14:43:51 srvlan dhcpd: Added reverse map from 32.3.168.192.in-addr.arpa. to debian11-vm1.intr...
```

Puis sur le terminal 2 pour le démon named :  
\- L'approbation de la clé tsig.key.  
\- L'existence de 3 adding an RR de type A, TXT et PTR.

```
Feb 21 14:43:51 srvlan named: client @0x7...24e8 192.168.3.1#6.../key tsig.key: signer "tsig.key" approved

Feb 21 14:43:51 srvlan named: client @0x7...24e8 192.168.3.1#6.../key tsig.key: updating zone 'intra.loupipfire.fr/IN': adding an RR at 'debian11-vm1.intra.loupipfire.fr' A 192.168.3.32

Feb 21 14:43:51 srvlan named: client @0x7...24e8 192.168.3.1#6.../key tsig.key: updating zone 'intra.loupipfire.fr/IN': adding an RR at 'debian11-vm1.intra.loupipfire.fr' TXT "3164da5294afbe..."

Feb 21 14:43:51 srvlan named: client @0x7...1a48 192.168.3.1#4.../key tsig.key: signer "tsig.key" approved

Feb 21 14:43:51 srvlan named: client @0x7...1a48 192.168.3.1#4.../key tsig.key: updating zone '3.168.192.in-addr.arpa/IN': deleting rrset at '32.3.168.192.in-addr.arpa' PTR

Feb 21 14:43:51 srvlan named: client @0x7...1a48 192.168.3.1#4.../key tsig.key: updating zone '3.168.192.in-addr.arpa/IN': adding an RR at '32.3.168.192.in-addr.arpa' PTR debian11-vm1.intr...
```

Accédez ensuite au dossier /var/lib/bind/, attendez quelques minutes et vérifiez dans le fichier DNS db.intra.loupipfire.fr.directe l'ajout automatique :  
\- d'enregistrements A et TXT pour debian11-vm1.

```
$ORIGIN .
$TTL 86400	; 1 day
intra.loupipfire.fr  IN SOA srvlan.intra.loup... (
	8          ; serial
	604800     ; refresh (1 week)
	84600      ; retry (23 hours)
	2419200    ; expire (4 weeks)
	604800     ; minimum (1 week)
	)
                     NS	srvlan.intra.loupipfire.fr.
$ORIGIN intra.loupipfire.fr.
$TTL 3600	; 1 hour
ctn1		        A	192.168.3.20
			TXT	"318b3ddc97..."
ctn2			A	192.168.3.31
			TXT	"31eefe30c9..."
debian11-vm1		A	192.168.3.32
			TXT	"3164da5294..."
debian11-vm2		A	192.168.3.33
			TXT	"316e74b10f..."
$TTL 86400	; 1 day
ovs			A	192.168.3.15
srvlan			A	192.168.3.1
```

Vérifiez dans le fichier DNS db.intra.loupipfire.fr.inverse l'ajout pour debian11-vm1 :  
\- d'un enregistrement PTR.

```
$ORIGIN .
$TTL 86400	; 1 day
3.168.192.in-addr.arpa IN SOA srvlan.intra.loup... (
		7          ; serial
		604800     ; refresh (1 week)
		86400      ; retry (1 day)
		2419200    ; expire (4 weeks)
		604800     ; minimum (1 week)
		)
		       NS srvlan.intra.loupipfire.fr.
$ORIGIN 3.168.192.in-addr.arpa.
1	PTR	srvlan.intra.loupipfire.fr.
15	PTR	ovs.intra.loupipfire.fr.
$TTL 3600	; 1 hour
20	PTR	ctn1.intra.loupipfire.fr.
31	PTR	ctn2.intra.loupipfire.fr.
32	PTR	debian11-vm1.intra.loupipfire.fr.
33	PTR	debian11-vm2.intra.loupipfire.fr.
```

Le bail attribué a été mis en cache dans le fichier /var/lib/dhcp/dhcpd.leases de la VM srvlan.

Effectuez à présent des pings entre srvlan et debian11-vm1 pour vérifier que la résolution DNS interne fonctionne de nouveau correctement.

#### _3.2 - Clients debian11-vm2, ovs, ctn1 et ctn2_

Idem client debian11-vm1, rebootez chacun d'eux et vérifiez la MAJ auto des fichiers de zone DNS ainsi que le fonctionnement correct de la résolution DNS interne entre toutes les VM et conteneurs LXC de la zone LAN.

### 4 - Bilan

Les échanges entre les serveurs DNS et DHCP, tous deux situés sur le même hôte, sont maintenant sécurisés par clé TSIG.

Cette sécurité peut évidemment être appliquée pour des serveurs distants l'un de l'autre.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà une bonne chose de faite.  
Le mémento 8.11 vous attend pour la  
mise en place d'un serveur Web PHP  
MySQL sur la VM srvdmz.

[Mémento 8.11](https://familleleloup.no-ip.org/site-web-php-mysql-debian11/)
