---
title: "DNS split / Debian 11"
date: "2022-06-21"
categories: 
  - "services-dns-split"
---

## Mémento 10.11 - DNS Split-Brain

Rappel, 2 entités distinctes coexistent sur le réseau :  
\- Domaine loupvirtuel.fr en zone interne publique.  
\- Sous-domaine intra.loupipfire.fr en zone interne privée.

Le domaine n'est actuellement pas joignable depuis Internet et le sous-domaine n'est pas compatible avec le domaine.

Pour le fun, vous allez créer un **DNS split** qui permettra de joindre le domaine depuis Internet ou la zone interne et une **délégation de zone** déclarant un sous-domaine compatible qui sera géré par srvlan.

2 serveurs DNS dont 1 à créer seront utilisés :  
\- 1 sur srvdmz pour le domaine loupvirtuel.fr _(création)_.  
\- 1 sur srvlan pour le sous-domaine intra.loupvirtuel.fr.

Vous finirez en mettant en place un **proxy inverse** qui permettra de joindre le site Web du sous-domaine depuis Internet.

### 1 - Préambule

#### _1.1 - Rôle du DNS split_

Fournir une IP publique aux requêtes DNS provenant de la zone Internet et une IP locale aux requêtes provenant des zones internes DMZ/LAN.

Cela évitera notamment de dévoiler au monde extérieur l'IP interne du domaine loupvirtuel.fr.

#### _1.2 - Rôle de la délégation de zone_

Déléguer la gestion du sous-domaine intra.louvirtuel.fr au serveur DNS de la zone LAN qui sera notamment, pour les noms d'hôtes des VM de la zone LAN, chargé de répondre aux requêtes DNS issues de la zone DMZ.

#### _1.3 - Rôle du proxy inverse_

Permettre à un utilisateur d'Internet d'accéder au serveur interne de la zone LAN. Placé en zone DMZ, il agira également comme une barrière de protection.

### 2 - Mise en place du service DNS sur srvdmz

#### _2.1 - Installation d'un serveur Bind_

Commencez par mettre à jour la distribution Debian :

\[srvdmz@srvdmz:~$\] sudo apt update 
\[srvdmz@srvdmz:~$\] sudo apt upgrade
\[srvdmz@srvdmz:~$\] sudo reboot

et installez le même serveur DNS que sur srvlan :

\[srvdmz@srvdmz:~$\] sudo apt install bind9

Le démon bind9 démarre automatiquement.  
\- La configuration de base a été créée dans /etc/bind/.  
\- Les fichiers de zone le seront dans /var/lib/bind/.

La synchronisation horaire normalement assurée par le service systemd-timesyncd de Debian est désactivée automatiquement au profit de celle assurée par le service vboxadd de VirtualBox.

Si vous n'utilisez pas VirtualBox, vérifiez ce service :

\[srvdmz@srvdmz:~$\] sudo systemctl status systemd-timesyncd

Celui-ci doit montrer un serveur x.debian.pool.ntp.org.

#### _2.2 - Création ACL, vues/zones de recherche_

Les Access Control List faciliteront l'écriture/lecture de la configuration en proposant de changer les définitions de réseaux IP filtrés par des alias plus explicites.

Les vues internes et externes contiendront pour chacune d'elles les définitions de leurs propres zones de recherche et fichiers de zone associés ainsi que celles des zones en délégation.

Les zones de recherche directe permettront de renvoyer une adresse IP associée à un nom alors que les zones de recherche inverse renverront un nom associé à une adresse IP.

Au préalable, éditez le fichier named.conf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/bind/named.conf

et commentez l'inclusion des zones par défaut :

#include "/etc/bind/named.conf.default-zones"; 

Editez ensuite le fichier named.conf.local :

\[srvdmz@srvdmz:~$\] sudo nano /etc/bind/named.conf.local

et créez les ACL, vues, zones de recherche et délégations en ajoutant ceci en fin de celui-ci :

\# Listes de contrôle d'accès pour les zones internes/externes
acl interne { 192.168.2.0/24; 192.168.4.0/24; };
acl externe { any; };

# Définition vue interne, zones loupvirtuel.fr/intra.loupvirtuel.fr
view "interne" {
match-clients { interne; };

# zone de recherche directe loupvirtuel.fr
zone "loupvirtuel.fr" IN {
type master;
file "/var/lib/bind/db.loupvirtuel.fr.directe";
allow-update { none; };
};

# zone de recherche inverse loupvirtuel.fr
zone "4.168.192.in-addr.arpa" IN {
type master;
file "/var/lib/bind/db.loupvirtuel.fr.inverse";
allow-update { none; };
};

# zone de recherche directe sous-domaine intra.loupvirtuel.fr
# **Zone déléguée** au serveur DNS srvlan
zone "intra.loupvirtuel.fr" IN {
type forward;
forward only;
forwarders { 192.168.3.1; };
};

# zone de recherche inverse sous-domaine intra.loupvirtuel.fr
# **Zone déléguée** au serveur DNS srvlan
zone "3.168.192.in-addr.arpa" IN {
type forward;
forward only;
forwarders { 192.168.3.1; };
};

# Réinsertion des zones par défaut fournies par bind9
include "/etc/bind/named.conf.default-zones";

};

# Définition vue externe, zone Internet > zone loupvirtuel.fr
view "externe" {
match-clients { externe; };

# zone de recherche directe loupvirtuel.fr
zone "loupvirtuel.fr" IN {
type master;
file "/var/lib/bind/db.loupvirtuel.fr.directe.externe";
allow-update { none; };
};

};

#### _2.3 - Création des fichiers de zone vue interne_

Créez le fichier pour la zone directe :

\[srvdmz@srvdmz:~$\] cd /var/lib/bind
\[srvdmz@srvdmz:~$\] sudo nano db.loupvirtuel.fr.directe

et insérez les lignes suivantes :

 ; DNS - Fichier de zone résolution directe pour loupvirtuel.fr
$TTL 86400
@ IN SOA srvdmz.loupvirtuel.fr. root.loupvirtuel.fr. (
2022062101
1w
1d
4w
1w )
@ IN NS srvdmz.loupvirtuel.fr.
srvdmz IN A 192.168.4.2
@ IN A 192.168.4.2
www IN CNAME srvdmz

; Déclaration du serveur gérant la zone déléguée intra.loupvir...
$ORIGIN intra.loupvirtuel.fr.
@ IN NS srvlan.intra.loupvirtuel.fr.
srvlan IN A 192.168.3.1                          

2022062101 = créé le 21/06/2022 \+ index de départ 01

Le fichier contient les enregistrements permettant d'accéder à la zone loupvirtuel.fr ainsi qu'une déclaration de délégation de zone pour l'accès au sous-domaine intra.loupvirtuel.fr.

Créez maintenant le fichier pour la zone inverse :

\[srvdmz@srvdmz:~$\] cd /var/lib/bind
\[srvdmz@srvdmz:~$\] sudo nano db.loupvirtuel.fr.inverse

et insérez les lignes suivantes :

; DNS - Fichier de zone résolution inverse pour loupvirtuel.fr
$TTL 86400
@ IN SOA srvdmz.loupvirtuel.fr. root.loupvirtuel.fr. (
2022062101
1w
1d
4w
1w )
@ IN NS srvdmz.loupvirtuel.fr.
2 IN PTR srvdmz.loupvirtuel.fr.

#### _2.4 - Création du fichier de zone vue externe_

Effectuez une copie du fichier db.loupvirtuel.fr.directe :

\[srvdmz@srvdmz:~$\] cd /var/lib/bind
\[srvdmz@srvdmz:~$\] sudo cp \*directe db.loupvirtuel.fr.directe.externe

et modifiez la copie avec l'éditeur nano comme suit :

; DNS - Fichier de zone résolution directe pour loupvirtuel.fr
$TTL 86400
@ IN SOA srvdmz.loupvirtuel.fr. root.loupvirtuel.fr. (
2022062101
1w
1d
4w
1w )
@ IN NS srvdmz.loupvirtuel.fr.
srvdmz IN A 192.168.x.w
@ IN A 192.168.x.w
www IN CNAME srvdmz

Remplacez 192.168.x.w par l'IP de la carte RED IPFire.  
N'oubliez pas de supprimer la partie zone déléguée.

#### _2.5 - Réglage des permissions sur les fichiers_

Attachez les 3 fichiers de zone créés au groupe bind :

\[srvdmz@srvdmz:~$\] sudo chgrp bind /var/lib/bind/\*loupvirtuel\*

Le démon bind9 pourra ainsi accéder à leur contenu.

Modifiez si nécessaire leur permission à la valeur 644 :

\[srvdmz@srvdmz:~$\] sudo chmod 644 /var/lib/bind/\*loupvirtuel\*

6 > Lecture/Ecriture pour le propriétaire root.  
4 > Lecture pour le groupe bind.  
4 > Lecture pour les autres utilisateurs.

#### _2.6 - Réglage des fichiers hosts et resolv.conf_

Avant, relancez bind9 pour traiter les fichiers ci-dessus :

\[srvdmz@srvdmz:~$\] sudo systemctl restart bind9
\[srvdmz@srvdmz:~$\] sudo systemctl status bind9

Editez ensuite le fichier DNS de nom hosts :

\[srvdmz@srvdmz:~$\] sudo cat /etc/hosts

et vérifiez qu'il contient bien le FQDN de srvdmz :

192.168.4.2 srvdmz.loupvirtuel.fr srvdmz loupvirtuel.fr 

Corrigez ou ajoutez la ligne si nécessaire.

Videz le contenu du fichier DNS /etc/resolv.conf puis désactivez son remplissage auto par le service de NetworkManager comme suit :

\[srvdmz@srvdmz:~$\] cd /etc/NetworkManager/conf.d
\[srvdmz@srvdmz:~$\] sudo nano 90-dns-none.conf

Entrez les 2 lignes ci-dessous :

\[main\]
dns=none

et relancez les services NetworkManager et bind9 :

\[srvdmz@srvdmz:~$\] sudo systemctl reload NetworkManager
\[srvdmz@srvdmz:~$\] sudo systemctl restart NetworkManager
\[srvdmz@srvdmz:~$\] sudo systemctl status NetworkManager

\[srvdmz@srvdmz:~$\] sudo systemctl restart bind9
\[srvdmz@srvdmz:~$\] sudo systemctl status bind9

Vérifiez que le fichier resolv.conf est bien toujours vide :

\[srvdmz@srvdmz:~$\] sudo nano /etc/resolv.conf

et si OK, entrez les lignes suivantes :

\# Fichier resolv.conf - Client DNS

# Nom du domaine local
domain loupvirtuel.fr

# Ajout automatique du nom de domaine aux hôtes
# non pleinement qualifiés
search loupvirtuel.fr
search intra.loupvirtuel.fr

# Adresses IP du ou des serveurs DNS à interroger
nameserver 192.168.4.2

#### _2.7 - Réglage de la résolution DNS externe_

Comme dans le [mémento 7.11](/dns-statique-debian11/#5_-_Declaration_dun_serveur_DNS_externe), vous allez faire en sorte que les demandes de résolution externe _(Ex : ping www.yahoo.fr)_ soient traitées par le serveur DNS de la box Internet.

Editez pour cela le fichier named.conf.options :

\[srvdmz@srvdmz:~$\] cd /etc/bind
\[srvdmz@srvdmz:~$\] sudo nano named.conf.options

et modifiez celui-ci afin qu'il contienne ces lignes :

options {
directory "/var/cache/bind";

# Pas d'utilisation du protocole DNSSEC
dnssec-validation no;

# Si réponse locale impossible, envoi des requêtes DNS
# vers la box Internet.
forward only;
forwarders { 192.168.x.z; };   \# Adresse IP de la box Internet
auth-nxdomain no;

listen-on-v6 { any; };
};

Remplacez 192.168.x.z par l'IP de votre box Internet.

#### _2.8 - Sécurisation des services fournis par Bind_

Editez de nouveau le fichier named.conf.options :

\[srvdmz@srvdmz:~$\] sudo nano named.conf.options

et ajoutez ce qui suit au dessus de \# Pas d'utilis... :

allow-recursion { 192.168.4.0/24;192.168.2.0/24; };
allow-query { any; };
allow-query-cache { 192.168.4.0/24;192.168.2.0/24; };
minimal-responses no;

# Pas d'utilisation du protocole DNSSEC \# ligne déjà existante

\- Récursivité limitée aux réseaux indiqués.  
\- Interrogation du DNS autorisée pour tous les réseaux.  
\- Accès au cache DNS limités aux réseaux indiqués.  
\- Réponses retournées complètes aux requêtes DNS .

La récursivité, par sécurité, est refusée pour les requêtes DNS provenant d'Internet.

#### _2.9 - Sécurisation des liaisons DNS avec TSIG_

La Transaction Signature permettra de sécuriser la liaison entre les serveurs DNS de srvdmz et srvlan.

Commencez par créer la clé secrète de TSIG :

\[srvdmz@srvdmz:~$\] cd /etc/bind
\[srvdmz@srvdmz:~$\] su root

\[root@srvdmz:~$\] sudo tsig-keygen -a HMAC-SHA256 tsig.key  > tsig-dns-split.key

\[root@srvdmz:~$\] chown bind:bind tsig-dns-split.key
\[root@srvdmz:~$\] chmod 640 tsig-dns-split.key
\[root@srvdmz:~$\] exit

La Cde tsig-keygen est fournie avec le paquet bind9.

Affichez le contenu de la clé tsig-dns-split.key :

\[srvdmz@srvdmz:~$\] sudo cat tsig-dns-split.key

Retour :

```
key "tsig.key" {
	algorithm hmac-sha256;
	secret "jXfcB8I4v9zVrhZADPcrICdunwzYcmtDo74+iS1DlJo=";
};
```

Notez la valeur de la clé pour l'utiliser ci-dessous.

Editez, pour déclarer l'usage de la clé, named.conf.local :

\[srvdmz@srvdmz:~$\] sudo nano /etc/bind/named.conf.local

et ajoutez ce contenu en fin de fichier :

\# Déclaration de la clé TSIG et de son algorithme de chiffrage
key tsig-dns-split.key.
{
algorithm hmac-sha256;
secret "jXfcB8I4v9zVrhZADPcrICdunwzYcmtDo74+iS1DlJo=";
};
# IP du serveur DNS en délégation pour intra.loupvirtuel.fr
server 192.168.3.1
{
keys { tsig-dns-split.key; };
};

_Nota : Ne pas oublier le point en fin de 2ème ligne._

Redémarrez le serveur DNS :

\[srvdmz@srvdmz:~$\] sudo systemctl restart bind9
\[srvdmz@srvdmz:~$\] sudo systemctl status bind9

Retour à observer :

```
● named.service - BIND Domain Name Server
   Loaded: loaded (/lib/sys...named.service; ena...)
   Active: active (running) since Thu 2022-06-23 ...
       Docs: man:named(8)
   Main PID: 6154 (named)
      Tasks: 6 (limit: 1116)
     Memory: 13.5M
        CPU: 102ms
     CGroup: /system.slice/named.service
             └─6154 /usr/sbin/named -f -u bind

juin.. managed-keys-zone/externe: loaded serial 5
juin.. zone 0.in-addr../IN/interne: load.. 1
juin.. zone 127.in-addr../IN/interne: load.. 1
juin.. zone 255.in-addr../IN/interne: load.. 1
juin.. zone loupvirtuel.fr/IN/in.. load.. 2022062101
juin.. zone 35.168.192.in-addr.../IN/interne: load..
juin.. zone localhost/IN/interne: loaded serial 2
juin.. zone loupvirtuel.fr/IN/ex.. load.. 2022062101
juin.. all zones loaded
juin.. running
```

Si tout est OK, arrêtez la VM :

\[srvdmz@srvdmz:~$\] sudo poweroff

### 3 - Modification du pare-feu IPFire

La suite du mémento impose l'ajout de nouvelles règles.

Accédez à l'interface Web d'IPFire depuis srvlan, puis :  
Menu IPFire > Pakfire  
Effectuez la mise à jour du système si nécessaire.

Ensuite :  
Menu Pare-feu > Options de pare-feu  
\> Afficher tous les réseaux... règles > on > Sauvegarder

Menu Système > Arrêter > Redémarrer  
  
Menu Pare-feu > Groupes de pare-feu  
\> Bouton Hôtes > Ajouter un hôte  
\> Nom : srvdmz > IP/MAC : 192.168.4.2  
\> Remarque : Serveur en zone orange > Sauvegarder  
  
\> Nom : srvlan > IP/MAC : 192.168.3.1  
\> Remarque : Serveur en zone verte > Sauvegarder  
\> Retour  
  
Menu Pare-feu > Règles de pare-feu  
\> Bouton Nouvelle règle  
  
a ) Ouverture du port HTTPS 443, Internet vers srvdmz :  
Nécessaire pour accéder à loupvirtuel.fr depuis Internet.  
  
Paramétrage :  
\- Source > Cochez Réseaux standards > ROUGE  
\- NAT > Cochez Utiliser la tradu... adresses réseau (NAT)  
\- Destination > Cochez Adresse IP de ... > 192.168.4.2  
\- Protocole > Sélectionnez TCP > Port de destina... > 443  
\- Paramètres additionnels > Remarque  
\> Entrez HTTPS vers serveur Web zone orange autorisé  
\> Ajouter > Appliquer les changements  
  
Recommencez comme ci-dessus pour le port HTTP 80. Cela permettra de vérifier la redirection de port vers HTTPS pour les URL http://www.loupvirtuel.fr et http://loupvirtuel.fr.

[![Capture - IPFire : HTTPS/HTTP ouvert zone Rouge vers Orange](/wp-content/uploads/2022/06/debian11-ipfire-dns-split-1-430x173.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/06/debian11-ipfire-dns-split-1.jpg)

IPFire : HTTPS/HTTP ouvert zone Rouge vers Orange

b ) Ouverture du port DNS 53, srvdmz vers srvlan :  
Indispensable pour le dialogue DNS entre les 2 serveurs mais représentant un petit trou de sécurité _(pinhole IPFire)_ sachant que les demandes de liaison de la zone orange vers la verte sont normalement toutes bloquées.  
  
Paramétrage :  
\- Source > Cochez Hôtes > srvdmz  
\- NAT > Cochez Utiliser la tradu... adresses réseau (NAT)  
\- Destination > Cochez Hôtes > srvlan  
\- Protocole > Sélectionnez TCP > Port de destina... > 53  
\- Paramètres additionnels > Remarques  
\> Entrez DNS port TCP srvdmz vers srvlan autorisé  
\> Ajouter > Appliquer les changements  
  
Recommencez comme ci-dessus pour le protocole UDP car le dialogue DNS se fait en priorité via ce protocole.

[![Capture - IPFire : DNS TCP/UDP ouvert zone Orange vers Verte](/wp-content/uploads/2022/06/debian11-ipfire-dns-split-2-430x217.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/06/debian11-ipfire-dns-split-2.jpg)

IPFire : DNS TCP/UDP ouvert zone Orange vers Verte

c ) Ouverture du port DNS 53, Internet vers srvdmz :  
Permettra d'interroger le serveur DNS depuis Internet.  
  
Paramétrage :  
\- Source > Cochez Réseaux standards > ROUGE  
\- NAT > Cochez Utiliser la tradu... adresses réseau (NAT)  
\- Destination > Cochez Adresse IP de … > 192.168.4.2  
\- Protocole > Sélectionnez UDP > Port de destina... > 53  
\- Paramètres additionnels > Remarque  
\> Entrez DNS port UDP Internet vers srvdmz autorisé  
\> Ajouter > Appliquer les changements

[![Capture - IPFire : DNS UDP ouvert zone Rouge vers Orange](/wp-content/uploads/2022/06/debian11-ipfire-dns-split-3-430x237.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/06/debian11-ipfire-dns-split-3.jpg)

IPFire : DNS UDP ouvert zone Rouge vers Orange

d ) Ouverture du port HTTP 80, srvdmz vers srvlan :  
Règle temporaire créée pour tester l'accès au serveur Web de srvlan depuis Internet ou srvdmz mais représentant un trou de sécurité entre la zone publique DMZ et la zone privée LAN.  
  
Paramétrage :  
\- Source > Cochez Hôtes > srvdmz  
\- NAT > Cochez Utiliser la tradu... adresses réseau (NAT)  
\- Destination > Cochez Hôtes > srvlan  
\- Protocole > Sélectionnez TCP > Port de destina... > 80  
\- Paramètres additionnels > Remarque  
\> Entrez HTTP vers serveur Web zone verte autorisé  
\> Ajouter > Appliquer les changements

[![Capture - IPFire : HTTP ouvert zone Orange vers Verte](/wp-content/uploads/2022/06/debian11-ipfire-dns-split-4-430x255.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/06/debian11-ipfire-dns-split-4.jpg)

IPFire : HTTP ouvert zone Orange vers Verte

### 4 - Modification du service DNS sur srvlan

#### _4.1 - Création des zones de recherche intra..._

Editez le fichier de configuration DNS named.conf.local :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf.local

et modifiez les 2 zones de recherche comme suit :

\# zone de recherche directe
zone "intra.loupvirtuel.fr" {
type master;
file "/var/lib/bind/db.intra.loupvirtuel.fr.directe";
allow-update { key "tsig.key"; };
};

# zone de recherche inverse
# Le réseau 192.168.3.0 aura pour adresse inverse
# 3.168.192.in-addr.arpa
zone "3.168.192.in-addr.arpa" {
type master;
file "/var/lib/bind/db.intra.loupvirtuel.fr.inverse";
allow-update { key "tsig.key"; };
};

#### _4.2 - Création des fichiers de zone intra..._

Créez le fichier pour la zone directe :

\[srvlan@srvlan:~$\] cd /var/lib/bind
\[srvlan@srvlan:~$\] sudo nano db.intra.loupvirtuel.fr.directe

et insérez les lignes suivantes :

;
; Zone DNS intra.loupvirtuel.fr - Fichier de zone pour
; la résolution directe
;
$TTL 86400
@ IN SOA srvlan.intra.loupvirtuel.fr. root.intra.loupvirtuel.fr. (
1 ; Serial
604800                ; Refresh - 1w (1 semaine)
84600                  ; Retry - 1d (1 jour)
2419200             ; Expire - 4w
604800 )             ; Negative Cache TTL - 1w
;
@ IN NS srvlan.intra.loupvirtuel.fr.
srvlan IN A 192.168.3.1
@ IN A 192.168.3.1
ovs IN A 192.168.3.15
www  IN CNAME srvlan

Puis créez le fichier pour la zone inverse :

\[srvlan@srvlan:~$\] sudo nano db.intra.loupvirtuel.fr.inverse

et insérez les lignes suivantes :

;
; Zone DNS intra.loupvirtuel.fr - Fichier de zone pour
; la résolution inverse
;
$TTL 86400
@ IN SOA srvlan.intra.loupvirtuel.fr. root.intra.loupvirtuel.fr. (
1
1w
1d
4w
1w )
;
@ IN NS srvlan.intra.loupvirtuel.fr.
1 IN PTR srvlan.intra.loupvirtuel.fr.
15 IN PTR ovs.intra.loupvirtuel.fr.

#### _4.3 - Réglage des permissions sur les fichiers_

\[srvlan@srvlan:~$\] sudo chgrp bind /var/lib/bind/\*loupvirtuel\*

\[srvlan@srvlan:~$\] sudo chmod 644 /var/lib/bind/\*loupvirtuel\*

#### _4.4 - Réglage des fichiers hosts et resolv.conf_

Editez le fichier DNS hosts :

\[srvlan@srvlan:~$\] sudo nano /etc/hosts

et modifiez son contenu comme suit :

127.0.0.1           localhost
127.0.1.1           srvlan

# IP de srvlan        + FQDN    + nom d'hôte     + domaine
192.168.3.1 srvlan.intra.loupvirtuel.fr  srvlan  intra.loupvirtuel.fr

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

Supprimez la ligne 192.168.4.2 loupvirtuel.fr, c'est le serveur DNS de srvdmz qui fournira dorénavant cette correspondance IP/Nom de domaine.

Videz le contenu du fichier DNS /etc/resolv.conf puis désactivez son remplissage auto par le service de NetworkManager comme suit :

\[srvlan@srvlan:~$\] cd /etc/NetworkManager/conf.d
\[srvlan@srvlan:~$\] sudo nano 90-dns-none.conf

Entrez les 2 lignes ci-dessous :

\[main\]
dns=none

et relancez les services NetworkManager et bind9 :

\[srvlan@srvlan:~$\] sudo systemctl reload NetworkManager
\[srvlan@srvlan:~$\] sudo systemctl restart NetworkManager
\[srvlan@srvlan:~$\] sudo systemctl status NetworkManager

\[srvlan@srvlan:~$\] sudo systemctl restart bind9
\[srvlan@srvlan:~$\] sudo systemctl status bind9

Vérifiez que le fichier resolv.conf est bien toujours vide :

\[srvlan@srvlan:~$\] sudo nano /etc/resolv.conf

et si OK entrez les lignes suivantes :

\# Fichier resolv.conf - Client DNS

# Nom du domaine local
domain intra.loupvirtuel.fr

# Ajout automatique du nom de domaine aux hôtes
# non pleinement qualifiés
search intra.loupvirtuel.fr

# Adresses IP du ou des serveurs DNS à interroger
nameserver 192.168.3.1

#### _4.5 - Modification des fichiers named.conf..._

Editez le fichier named.conf.options :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf.options

et modifiez la valeur de ces paramètres comme suit :

dnssec-validation no;
forwarders { 192.168.4.2; };   \# IP du serveur DNS srvdmz

Editez ensuite le fichier named.conf.local :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf.local

et ajoutez à la fin les informations sur la clé TSIG :

\# Déclaration de la clé TSIG à utiliser lors
# des transactions DNS entre srvdmz et srvlan
key tsig-dns-split.key.
{
algorithm hmac-sha256;
secret "jXfcB8I4v9zVrhZADPcrICdunwzYcmtDo74+iS1DlJo=";
};

# IP du serveur DNS srvdmz pour la zone "loupvirtuel.fr"
server 192.168.4.2
{
keys { tsig-dns-split.key; };
};

Editez le fichier de configuration DHCP dhcpd.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/dhcp/dhcpd.conf

et remplacez tous les loupipfire par loupvirtuel.

Redémarrez ensuite le service et vérifiez son statut :

\[srvlan@srvlan:~$\] sudo systemctl restart isc-dhcp-server
\[srvlan@srvlan:~$\] sudo systemctl status isc-dhcp-server

Purgez les Bdd contenant les baux générés par dhcpd :

\[srvlan@srvlan:~$\] cd /var/lib/dhcp
\[srvlan@srvlan:~$\] su root

\[root@srvlan:~#\] sudo rm dhcpd.leases~ \# Fichier Bdd archive
\[root@srvlan:~#\] echo "" > dhcpd.leases   \# Fichier Bdd exploité
\[root@srvlan:~#\] exit

Observez le contenu du cache DNS avant sa purge :

\[srvlan@srvlan:~$\] sudo rndc dumpdb -cache

La Cde ci-dessus crée dans /var/cache/bind/ un fichier de nom named\_dump.db. Celui-ci renferme le contenu du cache DNS qui peut ainsi être édité pour lecture.

\[srvlan@srvlan:~$\] sudo cat /var/cache/bind/named\_dump.db

Purgez maintenant le cache et relancez la même Cde :

\[srvlan@srvlan:~$\] sudo rndc flush
\[srvlan@srvlan:~$\] sudo rndc dumpdb -cache

Puis éditez et observez le contenu du nouveau cache.

Pour terminer, redémarrez la VM srvlan :

\[srvlan@srvlan:~$\] sudo reboot

#### _4.6 - Installation de Apache pour les tests_

Au préalable, redémarrez srvdmz sinon srvlan ne sera pas en mesure de joindre le monde extérieur pour notamment installer un nouveau paquet Linux.

Ensuite, installez le paquet Apache :

\[srvlan@srvlan:~$\] sudo apt install apache2
\[srvlan@srvlan:~$\] sudo systemctl status apache2

Erreur : AH00558: apache2: Could not reliably determine the server's fully qualified domain name

Entrez ces Cdes pour régler le problème du FQDN :

\[srvlan@srvlan:~$\] sudo echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/fqdn.conf

\[srvlan@srvlan:~$\] sudo a2enconf fqdn
\[srvlan@srvlan:~$\] sudo systemctl restart apache2
\[srvlan@srvlan:~$\] sudo systemctl status apache2

et depuis srvdmz, testez l'URL http://192.168.3.1.

Préparez la personnalisation de l'accueil du site Web :

\[srvlan@srvlan:~$\] cd /var/www/html
\[srvlan@srvlan:~$\] sudo mv index.html index.html.origine
\[srvlan@srvlan:~$\] sudo nano index.html

et remplissez le nouvel index.html avec ce contenu :

<!DOCTYPE html>
<html>
<head><meta charset="utf-8"></head>
<body style="background-color:#D8E6E7">

<h1 style="color:#000000">
Accueil Intranet de l'entreprise Loup Virtuel
</h1>

<p style="font-size:30px;color:#6E7783">
Ici, des outils pour vous aider dans votre travail !
</p>

</body>
</html>

Pour finir, testez de nouveau l'URL http://192.168.3.1.

### 5 - Modifications sur les VM de la zone LAN

\- VM ovs  
Editez son fichier résolveur DNS resolv.conf :

\[switch@ovs:~$\] sudo nano /etc/resolv.conf

et remplacez tout le contenu par les lignes suivantes :

\# Fichier resolv.conf - Client DNS

# Nom du domaine local
domain intra.loupvirtuel.fr

# Ajout automatique du nom de domaine aux hôtes
# non pleinement qualifiés
search intra.loupvirtuel.fr

# Adresses IP du ou des serveurs DNS à interroger
nameserver 192.168.3.1

Pour finir, redémarrez ovs et vérifiez la mise à jour automatique des fichiers DNS resolv.conf des conteneurs ctn1 et 2 _(Réf. [Mémento 5.12](/virtualbox-debian11-lxc-partie-1/))_.

\- VM debian11-vm\*  
Faites un clic droit sur l'applet réseau NetworkManager :  
\> Modifier les connexions...

Une fenêtre Connexions réseau s'ouvre :  
\> Sélectionnez la connexion affichée  
\> Cliquez sur l'icône roue dentée située en bas à gauche

Une fenêtre Modification de ... s'ouvre :  
\> Onglet Paramètres IPv4  
\> Domaines de recherche supplémentaires  
\> Remplacez intra.loupipfire.fr par intra.loupvirtuel.fr  
\> Enregistrer

Le paramètre search du fichier /etc/resolv.conf doit être mis à jour comme ceci :

\[client-linux@debian11-vm\*:~$\] sudo systemctl restart NetworkManager

\[client-linux@debian11-vm\*:~$\] cat /etc/resolv.conf

Par curiosité, observez au bout de quelques minutes la bonne MAJ dynamique du fichier de zone DNS /var/lib/bind/db.intra.loupvirtuel.fr.directe situé sur srvlan :

[![Capture - srvlan : zone directe intra.loupvirtuel.fr](/wp-content/uploads/2022/06/debian11-dns-spli-zone-directe-430x256.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/06/debian11-dns-spli-zone-directe.jpg)

srvlan : Fichier de zone db.intra.loupvirtuel.fr.directe

### 6 - Contrôle de la configuration DNS split

#### _6.1 - Côté srvdmz_

a ) Vérifiez l'usage de TSIG entre srvdmz/srvlan :

\[srvdmz@srvdmz:~$\] sudo dig -y hmac-sha256:tsig-dns-split.key:jXfcB8I4v9zVrhZADPcrICdunwzYcmtDo74+iS1DlJo= @192.168.3.1 AXFR intra.loupvirtuel.fr

Retour :

```
; <<>> DiG .. <<>> -y hmac-sha25.. intra.virtuel.fr
; (1 server found)
;; global options: +cmd
intra.loupvirt.. 86400 IN SOA srvlan.intra.loup..
intra.loupvirt.. 86400 IN NS  srvlan.intra.loup..
intra.loupvirt.. 86400 IN A   192.168.3.1
ctn1.intra.loupvirt.. 3600 IN TXT   "318b3ddc97.."
ctn1.intra.loupvirt.. 3600 IN A	    192.168.3.20
ctn2.intra.loupvirt.. 3600 IN TXT   "31eefe30c9.."
ctn2.intra.loupvirt.. 3600 IN A	    192.168.3.31
debian11-vm1.intra..  3600 IN TXT   "3164da5294a.."
debian11-vm1.intra..  3600 IN A	    192.168.3.32
debian11-vm2.intra..  3600 IN TXT   "316e74b10ff.."
debian11-vm2.intra..  3600 IN A	    192.168.3.33
ovs.intra.loupvirt.. 86400 IN A	    192.168.3.15
srvlan.intra.loupv.. 86400 IN A	    192.168.3.1
www.intra.loupvirt.. 86400 IN CNAME srvlan.intra.l..
intra.loupvirtuel..  86400 IN SOA   srvlan.intra.l..
tsig-dns-split.key.  0 ANY TSIG hmac... NOERROR 0 
;; Query time: 4 msec
;; SERVER: 192.168.3.1#53(192.168.3.1)
;; WHEN: Tue Jun 28 16:06:47 CEST 2022
;; XFR size: 15 records (messages 1, bytes 624)
```

b ) Vérifiez le contenu du fichier named.conf :

\[srvdmz@srvdmz:~$\] sudo named-checkconf

Aucune erreur ne sera retournée si OK.

c ) Vérifiez la zone directe db.loupvirtuel.fr.directe :

\[srvdmz@srvdmz:~$\] cd /var/lib/bind
\[srvdmz@srvdmz:~$\] sudo named-checkzone -d loupvirtuel.fr db.loupvirtuel.fr.directe

Retour :

```
loading "loupvirtuel.fr" from "..directe" class "IN"
zone loupvirtuel.fr/IN: loaded serial 2022062101
OK
```

d ) Vérifiez la zone inverse db.loupvirtuel.fr.inverse :

\[srvdmz@srvdmz:~$\] sudo named-checkzone -d 3.168.192.in-addr.arpa db.loupvirtuel.fr.inverse

Retour :

```
loa.. "3.168.192.in-add.." .. "..inverse" class "IN"
zone 3.168.192.in-add../IN: loaded serial 2022062101
OK
```

e ) Vérifiez le résultat des 2 Cdes suivantes :

\[srvdmz@srvdmz:~$\] host -v srvdmz.loupvirtuel.fr

Retour :

```
Trying "srvdmz.loupvirtuel.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id..
;; flags: qr aa rd ra; ..RY: 1, ..ER: 1, ..TY: 1,..

;; QUESTION SECTION:
;srvdmz.loupvirtuel.fr.	     IN A

;; ANSWER SECTION:
srvdmz.loupvirtuel.fr. 86400 IN A  192.168.4.2

;; AUTHORITY SECTION:
loupvirtuel.fr.	       86400 IN NS srvdmz.loupvir..

Received 69 bytes from 192.168.4.2#53 in 1 ms
Trying "srvdmz.loupvirtuel.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id..
;; flags: ..; ..RY: 1, ..ER: 0, ..TY: 1, ..AL: 0

;; QUESTION SECTION:
;srvdmz.loupvirtuel.fr.	     IN AAAA

;; AUTHORITY SECTION:
loupvirtuel.fr.	       86400 IN SOA srvdmz.loupvir..

Received 80 bytes from 192.168.4.2#53 in 1 ms
Trying "srvdmz.loupvirtuel.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id..
;; flags: ..; ..RY: 1, ..ER: 0, ..TY: 1, ..AL: 0 
;; QUESTION SECTION:
;srvdmz.loupvirtuel.fr.	     IN MX

;; AUTHORITY SECTION:
loupvirtuel.fr.	       86400 IN SOA srvdmz.loupvir..

Received 80 bytes from 192.168.4.2#53 in 0 ms
```

\[srvdmz@srvdmz:~$\] sudo dig SOA srvdmz.loupvirtuel.fr

Retour :

```
; <<>> DiG 9.16.27... <<>> SOA srvdmz.loupvirtuel.fr
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id..
;; flags: ..; ..RY: 1, ..ER: 0, ..TY: 1, ..AL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 411fb5402c2a5322532f434a5ef0c784.. (good)
;; QUESTION SECTION:
;srvdmz.loupvirtuel.fr.	       IN   SOA

;; AUTHORITY SECTION:
loupvirtuel.fr.	        86400  IN   SOA  srvdmz.lo..

;; Query time: 0 msec
;; SERVER: 192.168.4.2#53(192.168.4.2)
;; WHEN: Wed. jun 29 17:00:20 CEST 2022
;; MSG SIZE  rcvd: 119
```

f ) Contrôlez les IP retournées selon le domaine ciblé :

\[srvdmz@srvdmz:~$\] nslookup loupvirtuel.fr

Retour :

```
Server:		192.168.4.2
Address:	192.168.4.2#53

Name:	loupvirtuel.fr
Address: 192.168.4.2
```

\[srvdmz@srvdmz:~$\] nslookup intra.loupvirtuel.fr

Retour :

```
Server:		192.168.4.2
Address:	192.168.4.2#53

Non-authoritative answer:
Name:	intra.loupvirtuel.fr
Address: 192.168.3.1
```

#### _6.2 - Côté srvlan_

a ) Vérifiez l'usage de TSIG entre srvlan/srvdmz :

\[srvlan@srvlan:~$\] sudo dig -y hmac-sha256:tsig-dns-split.key:jXfcB8I4v9zVrhZADPcrICdunwzYcmtDo74+iS1DlJo= @192.168.4.2 AXFR loupvirtuel.fr

et réalisez les mêmes tests que ci-dessus avec srvdmz :

\[srvlan@srvlan:~$\] sudo named-checkconf

\[srvlan@srvlan:~$\] cd /var/lib/bind

\[srvlan@srvlan:~$\] sudo named-checkzone -d intra.loupvirtuel.fr db.intra.loupvirtuel.fr.directe

\[srvlan@srvlan:~$\] sudo named-checkzone -d 3.168.192.in-addr.arpa db.intra.loupvirtuel.fr.inverse

\[srvlan@srvlan:~$\] sudo host -v srvlan.intra.loupvirtuel.fr

\[srvlan@srvlan:~$\] sudo dig SOA srvlan.intra.loupvirtuel.fr

b ) Vérifiez ensuite le statut du serveur DHCP :

\[srvlan@srvlan:~$\] sudo systemctl status isc-dhcp-server

Redémarrez les VM debian11-vm\* et vérifiez sur srvlan que le fichier des baux /var/lib/dhcp/dhcpd.leases montre à présent le sous-domaine intra.loupvirtuel.fr.

c ) Contrôlez le résultat des pings suivants :

\[srvlan@srvlan:~$\] ping debian11-vm1
\[srvlan@srvlan:~$\] ping debian11-vm2.intra.loupvirtuel.fr
\[srvlan@srvlan:~$\] ping srvlan                                      \# 127.0.1.1
\[srvlan@srvlan:~$\] ping srvlan.intra.loupvirtuel.fr \# 192.168.3.1
\[srvlan@srvlan:~$\] ping ovs
\[srvlan@srvlan:~$\] ping ctn1.intra.loupvirtuel.fr
\[srvlan@srvlan:~$\] ping ctn2
\[srvlan@srvlan:~$\] ping intra.loupvirtuel.fr
\[srvlan@srvlan:~$\] ping loupvirtuel.fr
\[srvlan@srvlan:~$\] ping www.google.fr

Tous les pings doivent recevoir une réponse positive.

d ) Contrôlez les IP retournées selon le domaine ciblé :

\[srvlan@srvlan:~$\] nslookup intra.loupvirtuel.fr

Retour :

```
Server:		192.168.3.1
Address:	192.168.3.1#53

Name:	intra.loupvirtuel.fr
Address: 192.168.3.1
```

\[srvlan@srvlan:~$\] nslookup loupvirtuel.fr

Retour :

```
Server:	      192.168.3.1
Address:      192.168.3.1#53

Non-authoritative answer:
Name:	loupvirtuel.fr
Address: 192.168.4.2
```

#### _6.3 - Côté clients debian11-vm\*_

Testez les 2 Cdes suivantes :

\[client-linux@debian11-vm\*:~$\] dig SOA www.loupvirtuel.fr
\[client-linux@debian11-vm\*:~$\] dig SOA www.yahoo.fr 

Retours à observer dans la section AUTHORITY :  
\- SOA srvdmz.loupvirtuel.fr pour la première ligne.  
\- SOA yf1.yahoo.com pour la seconde.

Testez également la résolution directe et inverse :

\[client-linux@debian11-vm\*:~$\] nslookup www.loupvirtuel.fr
\[client-linux@debian11-vm\*:~$\] nslookup 192.168.4.2 

\- Cde 1, srvlan renvoie l'IP locale de srvweb.  
\- Cde 2, srvlan renvoie l'hôte srvdmz.loupvirtuel.fr.

### 7 - Test depuis Internet (simulation)

Le domaine loupvirtuel.fr n'étant pas public, vous ne pouvez pas accéder au site Web depuis Internet en tapant simplement son URL dans le champ adresse d'un navigateur Web.

En revanche, vous pouvez simuler un accès Internet depuis le PC hôte de VirtualBox.

#### _7.1 - Préparation pour la simulation_

L'exemple ci-dessous concerne un PC Windows 10/11.

Désactivez l'IPv6 et modifiez l'IPv4 du serveur DNS utilisé en remplaçant celle-ci par l'IP 192.168.x.w de la carte RED d'IPFire comme suit :

a) Désactivation du protocole IPv6 :

\- Clic droit sur le menu Windows 10/11 > Exécuter  
\> Entrez ncpa.cpl > OK  
\> Sélectionnez votre carte réseau active  
\> Clic droit sur celle-ci > Propriétés  

![Capture - Windows : IPv4/IPv6 - IPv6 à désactiver pour la simulation ](/wp-content/uploads/2020/06/dns-split-config-ip-serveur-dns_2.jpg)

Windows : IPv4/IPv6 - IPv6 à désactiver pour la simulation

b) Adresse IPv4 du serveur DNS préféré à utiliser :

\- Sélectionnez Protocole Internet version 4 (TCP/IPv4)  
\> Bouton Propriétés

![Capture : Windows : IPv4 - DNS préféré > 192.168.x.w = IP RED IPFire](/wp-content/uploads/2020/06/dns-split-config-ip-serveur-dns_1.jpg)

Windows : IPv4 - DNS préféré > 192.168.x.w _(carte RED IPFire)_

Vous pouvez à présent effectuer les tests ci-dessous.

#### _7.2 - Test du DNS de srvdmz depuis Internet_

Depuis l'Invite de commandes du PC Windows, entrez :

\[C:\\Users\\...>\] nslookup loupvirtuel.fr

Retour :

```
Serveur :   UnKnown
Address:  192.168.x.w

Nom :    loupvirtuel.fr  # Réponse du DNS de srvdmz
Address:  192.168.x.w    # IP externe retournée
```

\[C:\\Users\\...>\] nslookup yahoo.fr

Retour :

```
Serveur :   UnKnown
Address:  192.168.x.w

*** UnKnown ne parvient ... yahoo.fr : Query refused
```

\[C:\\Users\\...>\] nslookup intra.loupvirtuel.fr

Retour :

```
Serveur :   UnKnown
Address:  192.168.x.w

*** UnKnown ... intra.loupvirtuel.fr : Non-existe...
```

Les 2 dernières requêtes sont rejetées car, voir plus haut, la récursivité DNS a été bloquée pour les requêtes DNS provenant d'Internet, ceci pour raison de sécurité.

192.168.x.w = IP carte RED du serveur IPFire.

#### _7.3 - Test du site loupvirtuel.fr depuis Internet_

Testez depuis le navigateur Web du PC Windows les 2 URL suivantes :  
http://www.loupvirtuel.fr et http://loupvirtuel.fr

Toutes les deux doivent basculer vers le protocole https sous réserve d'avoir au préalable importé le certificat ssl loupvirtuel-ca.pem dans le magasin des autorités gérées par le navigateur Web _(voir [ici](/https-dotclear-debian11/#53_-_Importation_du_certificat_CA_dans_Firefox))_.

Les autres URL comme par exemple http://google.fr doivent rester sans réponse car la récursivité DNS est bloquée.

#### _7.4 - Test du site Web de srvlan depuis Internet_

Possible en utilisant le proxy inverse fourni par Apache.

Chargez les modules concernant la fonction de proxy :

\[srvdmz@srvdmz:~$\] sudo a2enmod proxy
\[srvdmz@srvdmz:~$\] sudo a2enmod proxy\_http

Editez ensuite le virtualhost du domaine loupvirtuel.fr :

\[srvdmz@srvdmz:~$\] cd /etc/apache2/sites-available
\[srvdmz@srvdmz:~$\] sudo nano loupvirtuel.conf

et ajoutez la section VirtualHost ci-dessous :

<VirtualHost \*:80>
ServerName extra.loupvirtuel.fr

<Proxy \*>
Order deny,allow
Allow from all
</Proxy>

ProxyRequests Off
ProxyPreserveHost On
ProxyPass "/" "http://192.168.3.1:80/"
ProxyPassReverse "/" "http://192.168.3.1:80/"

ErrorLog /var/log/apache2/extra.loupvirtuel.error.log
CustomLog /var/log/apache2/extra.loupvirtuel.access.log combined
</VirtualHost>

Redémarrez Apache :

\[srvdmz@srvdmz:~$\] sudo systemctl restart apache2
\[srvdmz@srvdmz:~$\] sudo systemctl status apache2

Modifiez maintenant le fichier de zone DNS suivant :

\[srvdmz@srvdmz:~$\] cd /var/lib/bind
\[srvdmz@srvdmz:~$\] sudo nano db.loupvirtuel.fr.directe.externe

en ajoutant l'enregistrement DNS ci-dessous :

extra IN A 192.168.x.w      \# IP carte RED de IPFire

Pour finir, redémarrer le serveur DNS :

\[srvdmz@srvdmz:~$\] sudo systemctl restart bind9

et testez, depuis le navigateur Web du PC hôte, l'URL :  
http://extra.loupvirtuel.fr.

Vous devez observer l'accueil du site Web de srvlan.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà, c'est terminé pour le DNS split.  
Le mémento 11.11 vous attend pour  
mettre en place un serveur de  
courrier basé sur Postfix.

[Mémento 11.1](https://familleleloup.no-ip.org/postfix-debian11/)
