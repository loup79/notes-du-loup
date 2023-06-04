---
title: "DNS dynamique / Debian 10"
date: "2022-04-22"
categories: 
  - "archives"
---

## Mémento 7.3 - Maj du DNS via DHCP

Le DNS statique installé sur la VM srvlan deviendra dynamique via DHCP.

### 1 - Préambule

Toutes les VM doivent être démarrées sauf srvdmz.

#### _1.1 - Rôle du service DNS dynamique via DHCP_

Modifier en temps réel les fichiers de zones DNS via le service DHCP en utilisant les démons named de bind9 et dhcpd de isc-dhcp-server.

### 2 - Configuration du DNS dynamique

#### _2.1 - Configuration initiale_

Vous avez, lors de la configuration DHCP du [mémento 7.2](/dhcp-isc-dhcp-server/), supprimé volontairement les enregistrements des VM debian10-vm\* dans les fichiers de zones DNS.

La résolution DNS interne est donc cassée :

\[srvlan@srvlan:~$\] ping debian10-vm1

Retour :

```
ping: debian10-vm1: Échec temporaire dans la résolution du nom
```

\[srvlan@srvlan:~$\] ping debian10-vm2.intra.loupipfire.fr 

Retour :

```
ping: debian10-vm2.intra.loupipfire.fr: Nom ou service inconnu
```

Seule la résolution DNS externe fonctionne :

\[srvlan@srvlan:~$\] ping yahoo.fr

Retour :

```
PING yahoo.fr (212.82.100.151) 56(84) bytes of data.
64 bytes from w2.src1.vip.ir2.yahoo.com (212.82.1...
64 bytes from w2.src1.vip.ir2.yahoo.com (212.82.1...
64 bytes from w2.src1.vip.ir2.yahoo.com (212.82.1...
64 bytes from w2.src1.vip.ir2.yahoo.com (212.82.1...
--- yahoo.fr ping statistics ---
4 packets transmitted, 4 received, 0% packet loss...
rtt min/avg/max/mdev = 34.618/35.337/37.056/1.01...
```

#### _2.2 - Création d'une clé secrète TSIG_

TSIG _(Transaction Signature)_ est un mécanisme permettant de sécuriser la communication avec un serveur DNS, ceci à base d'une clé secrète partagée.

Il peut être utilisé pour authentifier la source des requêtes et réponses DNS échangées lors d'une MAJ dynamique des fichiers de zones DNS via DHCP.

Cela dit, afin de sécuriser la liaison entre les serveurs DNS et DHCP, commencez par générer une clé reposant sur TSIG.

Entrez pour cela les Cdes suivantes :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo dnssec-keygen -a HMAC-MD5 -b 512 -n USER dhcp-rndc-key 

Pour information, l'utilitaire dnssec-keygen est issu du paquet bind9utils.

2 fichiers ont été générés :  
\- Kdhcp-rndc-key.+157+55955.key  
\- Kdhcp-rndc-key.+157+55955.private

Affichez le contenu du fichier \*.private :

\[srvlan@srvlan:~$\] sudo cat \*.private   

et sauvegardez temporairement la valeur de la clé TSIG :

```
Private-key-format: v1.3
Algorithm: 157 (HMAC_MD5)
Key: ExWb3EVRwELn... TGrZ6AxFmS.../lWmuX73g==
Bits: AAA=
Created: 20191106115054
Publish: 20191106115054
Activate: 20191106115054
```

Créez ensuite un fichier dhcp-rndc.key comme suit :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo touch dhcp-rndc.key
\[srvlan@srvlan:~$\] sudo chown root:bind dhcp-rndc.key
\[srvlan@srvlan:~$\] sudo chmod 640 dhcp-rndc.key   

Editez celui-ci :

\[srvlan@srvlan:~$\] sudo nano dhcp-rndc.key 

et entrez les lignes suivantes :

key "dhcp-rndc-key" {
     algorithm hmac-md5;
     secret "ExWb3EVRwELn… TGrZ6AxFmS…/lWmuX73g==";
 };

L'élément secret doit contenir la valeur de la clé TSIG.

Vous auriez pu aussi créer un fichier dhcp-rndc.key prérempli avec sa clé TSIG comme ceci :

\[srvlan@srvlan:~$\] cd /etc/bind

\[srvlan@srvlan:~$\] sudo tsig-keygen -a HMAC-SHA256 dhcp-rndc-key >> dhcp-rndc.key

\[srvlan@srvlan:~$\] sudo chown root:bind dhcp-rndc.key
\[srvlan@srvlan:~$\] sudo chmod 640 dhcp-rndc.key   

L'utilitaire tsig-keygen est issu du paquet bind9. 
L'algorithme SHA256 offre plus de sécurité que MD5.

#### _2.3 - Configuration côté serveur DNS_

Editez le fichier named.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf 

et ajoutez les 3 lignes suivantes en fin de celui-ci :

\# Déclaration de la clé TSIG qui sera utilisée pour
# la mise à jour dynamique du DNS
include "/etc/bind/dhcp-rndc.key"; 

Editez ensuite le fichier named.conf.local :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf.local

puis modifiez la zone de recherche directe comme suit :

file "/var/cache/bind/db.intra.loupipfire.fr.directe";
allow-update {key "dhcp-rndc-key";};

et la zone de recherche inverse comme ci-dessous :

file "/var/cache/bind/db.intra.loupipfire.fr.inverse";
allow-update {key "dhcp-rndc-key";}; 

Pour finir, créez les 2 liens symboliques suivants :

\[srvlan@srvlan:~$\] cd /var/cache/bind

\[srvlan@srvlan:~$\] sudo ln -s /etc/bind/db.intra.loupipfire.fr.directe .

\[srvlan@srvlan:~$\] sudo ln -s /etc/bind/db.intra.loupipfire.fr.inverse . 

Explications :  
La MAJ des fichiers de zones fait appel à l'utilisateur bind qui ne dispose pas du droit d'écriture dans /etc/bind/ d'où la création des liens ci-dessus.  
  
Le paramètre allow-update autorise les MAJ DNS si authentifiées avec la clé dhcp-rndc-key.

#### _2.4 - Configuration côté serveur DHCP_

Le service DHCP doit utiliser la même clé TSIG que le service DNS.

Pour cela, copiez le fichier dhcp-rndc.key dans /etc/dhcp/ et gérez ses permissions :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo cp dhcp-rndc.key /etc/dhcp/

\[srvlan@srvlan:~$\] cd /etc/dhcp
\[srvlan@srvlan:~$\] sudo chown root:root dhcp-rndc.key
\[srvlan@srvlan:~$\] sudo chmod 640 dhcp-rndc.key 

Editez ensuite le fichier dhcpd.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/dhcp/dhcpd.conf

et modifiez son contenu comme indiqué ci-dessous :

\# dhcpd.conf

# # Déclarations optionnelles :
# Domaine que les clients doivent utiliser pour résoudre
# les noms d'hôte via le DNS.
option domain-name "intra.loupipfire.fr";

# Liste des serveurs DNS disponibles pour les clients.
option domain-name-servers srvlan.intra.loupipfire.fr;

# Durée du bail par défaut si le client ne spécifie pas
# une durée (Ex: 86400 secondes = 1 jour).
default-lease-time 86400;

# Durée maxi du bail que peut demander le client
# (Ex: 604800 secondes = 7 jours).
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

# Envoi des logs sur un fichier autre que
# le fichier /var/log/syslog.
log-facility local7;

# Déclaration basique du sous-réseau local à traiter :
# Sous-réseau et masque de sous-réseau.
subnet 192.168.3.0 netmask 255.255.255.0 {

# Etendue de la plage DHCP.
range 192.168.3.30 192.168.3.50;

# Passerelle par défaut.
option routers srvlan.intra.loupipfire.fr;

\# Informations sur le serveur DNS à mettre à jour
# dynamiquement.
include "/etc/dhcp/dhcp-rndc.key";

zone intra.loupipfire.fr. {
       primary srvlan.intra.loupipfire.fr;
       key dhcp-rndc-key;
}

zone 3.168.192.in-addr.arpa. {
       primary srvlan.intra.loupipfire.fr;
       key dhcp-rndc-key;
} 
}

Enfin, relancez dans l'ordre les services DNS et DHCP :

\[srvlan@srvlan:~$\] sudo systemctl restart bind9
\[srvlan@srvlan:~$\] sudo systemctl restart isc-dhcp-server 

### 3 - Contrôle de bon fonctionnement

#### _3.1 - Client debian10-vm1_

Au préalable, ouvrez 2 terminaux sur le bureau de srvlan.

Connectez-vous ensuite sur ceux-ci en tant qu'utilisateur root _(Cde su root)_.

Entrez sur le terminal 1 cette demande de traçage :

\[root@srvlan:~#\] tail -f /var/log/dhcp/isc-dhcp-server.log

et sur le terminal 2 celle ci-dessous :

\[root@srvlan:~#\] tail -f /var/log/syslog

Redémarrez maintenant la VM debian10-vm1.

Vérifiez ensuite sur le terminal 1 pour le démon dhcpd :  
\- La présence de la mention Added new forward map.  
\- La présence de la mention Added reverse map.

```
17:19:35 srvlan dhcpd: DHCPREQUEST for 192.168.3.30 from 08:00... via enp0s8
17:19:35 srvlan dhcpd: DHCPACK on 192.168.3.30 to 08:00... (debian10-vm1) via enp0s8
17:19:35 srvlan dhcpd: Added new forward map from debian10-vm1.intra... to 192.168.3.30
17:19:35 srvlan dhcpd: Added reverse map from 30.3.168.192.in-addr.arpa. to debian10-...
```

Puis sur le terminal 2 pour le démon named :  
\- L'approbation de la clé dhcp-rndc-key.  
\- L'existence de 3 adding an RR types A, TXT et PTR.

```
17:19:35 srvlan named: client @0xb378... 192.168.3.1#54075/key dhcp-rndc-key... approved
17:19:35 srvlan named: client @0xb378... 192.168.3.1#54075/key dhcp-rndc-key: updating zone 'intra.loup.../IN': adding an RR at 'debian10-vm1.intra.loup...' A 192.168.3.30
17:19:35 srvlan named: client @0xb378... 192.168.3.1#54075/key dhcp-rndc-key: updating zone 'intra.loup.../IN': adding an RR at 'debian10-vm1.intra.loup...' TXT "00143e9a..."
17:19:35 srvlan named: client @0xb299... 192.168.3.1#35949/key dhcp-rndc-key... approved
17:19:35 srvlan named: client @0xb299... 192.168.3.1#35949/key dhcp-rndc-key: updating zone '3.168.192.in-addr.arpa/IN': deleting rrset at '30.3.168.192.in-addr.arpa' PTR
17:19:35 srvlan named: client @0xb299... 192.168.3.1#35949/key dhcp-rndc-key: updating zone '3.168.192.in-addr.arpa/IN': adding an RR at '30.3.168.192.in-addr.arpa' PTR debian10-vm1.intra.loupipfire.fr.
```

Accédez à présent au dossier /var/cache/bind/.

Vérifiez dans le fichier DNS db.intra.loupipfire.fr.directe l'ajout pour debian10-vm1 :  
\- d'enregistrements de type A et TXT.

```
$ORIGIN .
$TTL 86400	; 1 day
intra.loupipfire.fr IN SOA  srvlan.intra.loup... (
		2          ; serial
		604800     ; refresh (1 week)
		84600      ; retry (23 hours 30 ...)
		2419200    ; expire (4 weeks)
		604800     ; minimum (1 week)
		)
		    NS   srvlan.intra.loupipfire.fr.
$ORIGIN intra.loupipfire.fr.
$TTL 3600	; 1 hour
debian10-vm1    A	192.168.3.30
		TXT	"00143e9ad6e1779821deeafc..."
debian10-vm2	A	192.168.3.31
		TXT	"002000ed15e24cd6ac77dc03..."
$TTL 86400	; 1 day
ovs		A	192.168.3.15
srvlan		A	192.168.3.1
```

Vérifiez dans le fichier DNS db.intra.loupipfire.fr.inverse l'ajout pour debian10-vm1 :  
\- d'un enregistrement PTR.

```
$ORIGIN .
$TTL 86400	; 1 day
3.168.192.in-addr.arpa	IN SOA	srvlan.intra.lou... (
		2          ; serial
		604800     ; refresh (1 week)
		86400      ; retry (1 day)
		2419200    ; expire (4 weeks)
		604800     ; minimum (1 week)
		)
		NS	srvlan.intra.loupipfire.fr.
$ORIGIN 3.168.192.in-addr.arpa.
1		PTR	srvlan.intra.loupipfire.fr.
15		PTR	ovs.intra.loupipfire.fr.
$TTL 3600	; 1 hour
30      	PTR	debian10-vm1.intra.loup...
31		PTR	debian10-vm2.intra.loup...
```

Comparez par curiosité les contenus des 2 fichiers de zones DNS, ci-dessus modifiés dynamiquement, avec ceux d'origine situés dans /etc/bind/.

Le bail attribué a été mis en cache dans le fichier /var/lib/dhcp/dhcpd.leases de la VM srvlan.

Effectuez à présent des pings entre srvlan et debian10-vm1 pour vérifier que la résolution DNS interne fonctionne de nouveau correctement.

#### _3.2 - Client debian10-vm2_

Idem client debian10-vm1, en vérifiant une dernière fois que la résolution DNS interne fonctionne correctement entre toutes les VM de la zone LAN.

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Voilà une bonne chose de faite.  
Le mémento 8.1 vous attend pour la  
mise en place d'un serveur Web PHP  
MySQL sur la VM srvdmz.

[Mémento 8.1](/site-web-php-mysql/)
