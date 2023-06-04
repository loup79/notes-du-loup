---
title: "DNS statique / Debian 10"
date: "2022-04-24"
categories: 
  - "archives"
---

## Mémento 7.1 - DNS avec BIND 9

Le service DNS statique reposera sur bind 9 et sera installé sur srvlan pour gérer la zone LAN.

### 1 - Préambule

Le service permettra de créer un sous-domaine de loupipfire.fr de nom intra.loupipfire.fr et d'associer le nom de ce sous-domaine avec les adresses IP des VM situées en zone LAN.

Au préalable, mettez à jour vos systèmes Debian 10.

Pour cela, vérifiez le fichier /etc/apt/sources.list :

\# Debian Buster, mises à jour principales
deb http://deb.debian.org/debian/ buster main
deb-src http://deb.debian.org/debian/ buster main

# Debian Buster, mises à jour de sécurité
deb http://deb.debian.org/debian-security/ buster/updates main
deb-src http://deb.debian.org/debian-security/ buster/updates main

# Debian Buster,  mises à jour volatiles
deb http://deb.debian.org/debian/ buster-updates main
deb-src http://deb.debian.org/debian/ buster-updates main

# Debian Buster, mises à jour des paquets encore en test
deb http://deb.debian.org/debian/ buster-proposed-updates main

puis lancez les Cdes suivantes :

$ sudo apt update
$ sudo apt upgrade
$ sudo apt dist-upgrade

#### _1.1 - Configuration actuelle de la zone LAN_

a) Une IP fixe et un nom d'hôte pour chaque VM.  
Ex : IP fixe 192.168.3.2 pour l'hôte debian10-vm1

Cde utile pour découvrir les adresses IP :

\[user@hostname:~$\] ip address

Cdes utiles pour découvrir les noms d'hôtes :

\[user@hostname:~$\] cat /etc/hostname
\[user@hostname:~$\] cat /etc/hosts

b) Une Box Internet déclarée comme serveur DNS.

Cde utile pour découvrir les serveurs DNS exploités :

\[user@hostname:~$\] cat /etc/resolv.conf

c) Des pings qui émis depuis le LAN montrent un retour :  
\- Positif sur un nom de domaine Internet  
Ex : ping www.google.fr

\- Positif sur leur propre nom d'hôte  
Ex : ping debian10-vm1 depuis la VM debian10-vm1

\- Négatif sur les autres noms d'hôtes de la zone LAN

Exemple de retour :

```
client-linux@debian10-vm1:~$ ping srvlan
ping: srvlan: Aucune adresse associée ... l'hôte
client-linux@debian10-vm1:~$ 
```

Un serveur DNS local solutionnera le retour négatif.

### 2 - Mise en place du service DNS statique

Le système fournit bind et unbound soit les 2 serveurs DNS les plus utilisés sous Debian.

Le choix se portera sur bind qui contrairement à unbound peut être utilisé à la fois comme serveur de noms récursif et serveur de noms faisant autorité.  
  
Un serveur DNS récursif recherchera le résultat d'une requête DNS dans son cache ou à défaut interrogera un serveur DNS faisant autorité pour obtenir le résultat.  
  
Un serveur DNS faisant autorité contiendra le résultat, il n'interrogera pas d'autres serveurs et sera l’autorité finale contenant tous les noms d'hôtes et adresses IP d'une zone donnée.

Le service sera statique car les données seront renseignées manuellement.

#### _2.1 - Installation de bind9_

Installez le paquet bind9 :

\[srvlan@srvlan:~$\] sudo apt install bind9

et vérifiez le démarrage de celui-ci :

\[srvlan@srvlan:~$\] systemctl status bind9

Retour :

```
● bind9.service - BIND Domain Name Server
 Loaded: loaded (/lib/syst.../bind9.service; ...
 Active: active (running) since Mon 2019-08...
    Docs: man:named(8)
 Process: 1015 ExecStart=/usr/sbin/named ... SUC...
Main PID: 1016 (named)
   Tasks: 4 (limit: 679)
  Memory: 9.4M
   CGroup: /system.slice/bind9.service
      └─1016 /usr/sbin/named -u bind
```

Vérifiez son lancement automatique au boot de la VM :

\[srvlan@srvlan:~$\] systemctl is-enabled bind9

Retour :

```
enabled 
```

Vérifiez la version installée :

\[srvlan@srvlan:~$\] sudo named -v    

Retour :

```
BIND 9.11.5...Debian (Extended Supp...) <id:...> 
```

Vérifiez l'utilisation des ports TCP/UDP 53 et 953 :

\[srvlan@srvlan:~$\] sudo netstat -lnptu | grep named 

Retour :

```
tcp      0     0  192.168.3.1:53    .../named
tcp      0     0  192.168.2.2:53    .../named
tcp      0     0  127.0.0.1:53      .../named            
tcp      0     0  127.0.0.1:953     .../named          
tcp6     0     0  :::53             .../named          
tcp6     0     0  ::1:953           .../named          
udp      0     0  192.168.3.1:53    .../named          
udp      0     0  192.168.2.2:53    .../named          
udp      0     0  127.0.0.1:53      .../named          
udp      0     0  :::53             .../named
```

53 = Port DNS TCP/UDP par défaut  
953 = Port des Cdes rndc pour gérer le démon named

Vérifiez l'activation de l'outil associé rndc :

\[srvlan@srvlan:~$\] sudo rndc status 

Retour :

```
version: BIND 9.11.5-P4-5.1-Debian (Extended Supp...
running on srvlan: Linux i686 4.19.0-5-686-pae #...
boot time: Tue, 27 Aug 2019 13:49:48 GMT
last configured: Tue, 27 Aug 2019 13:49:48 GMT
configuration file: /etc/bind/named.conf
CPUs found: 1
worker threads: 1
UDP listeners per interface: 1
number of zones: 104 (97 automatic)
debug level: 0
xfers running: 0
xfers deferred: 0
soa queries in progress: 0
query logging is OFF
recursive clients: 0/900/1000
tcp clients: 4/150
server is up and running
```

L'interface en ligne de Cde rndc permet de gérer le démon named localement ou à distance.

Pour finir, observez les fichiers de configuration présents dans les dossiers suivants :

\[srvlan@srvlan:~$\] ls /etc/bind
\[srvlan@srvlan:~$\] ls /var/cache/bind   

### 3 - Configuration de base

#### _3.1 - Définition des zones de recherche_

Une zone de recherche directe permet à un service DNS interrogé de renvoyer une adresse IP associée à un nom alors qu'une zone de recherche inverse fera le contraire.  
  
Le fichier named.conf.local contient la configuration locale du serveur DNS bind9, vous y déclarerez donc les zones de recherche pour intra.loupipfire.fr.

Editez le fichier de configuration named.conf.local :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf.local

et ajoutez les lignes suivantes à la fin de celui-ci :

\# Zone ou Domaine intra.loupipfire.fr
# Définition des zones de recherche directe et inverse
# Serveur DNS maître pour le domaine _(type)_
# Fichiers de zones associés dans /etc/bind _(file)_
# MAJ dynamique des fichiers de zones > none _(allow-update)_  

# zone de recherche directe
zone "intra.loupipfire.fr" IN {
type master;
file "/etc/bind/db.intra.loupipfire.fr.directe";
allow-update { none; };
};

# zone de recherche inverse
# Le réseau 192.168.3.0 aura pour adresse inverse 
# 3.168.192.in-addr.arpa 
zone "3.168.192.in-addr.arpa" IN {
type master;
file "/etc/bind/db.intra.loupipfire.fr.inverse";
allow-update { none; };
};

#### _3.2 - Construction des zones de recherche_

Les fichiers de zones de recherche contiendront les directives et enregistrements de ressources pour le domaine intra.loupipfire.fr.

Créez le fichier pour la zone directe :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo nano db.intra.loupipfire.fr.directe

et insérez les lignes suivantes :

;
; DNS - Fichier de zone pour la résolution directe
;
$TTL 86400
@ IN SOA srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. (
1                             ; Serial
604800                  ; Refresh - 1w (1 semaine)
84600                    ; Retry - 1d (1 jour)
2419200               ; Expire - 4w 
604800 )               ; Negative Cache TTL - 1w
;
@ IN NS srvlan.intra.loupipfire.fr.
srvlan IN A 192.168.3.1
ovs IN A 192.168.3.15
debian10-vm1 IN A 192.168.3.2
debian10-vm2 IN A 192.168.3.4

\- Détail des paramètres - 
a) Directives SOA _(Start of Authority)_ :

<table class="has-fixed-layout"><tbody><tr><td>@</td><td class="has-text-align-left" data-align="left">Représente le nom de domaine de la zone soit <mark style="background-color:var(--base-3)" class="has-inline-color has-global-color-8-color">intra.loupipfire.fr</mark></td></tr><tr><td>IN SOA</td><td class="has-text-align-left" data-align="left">Désigne <mark style="background-color:var(--base-3)" class="has-inline-color has-global-color-10-color">srvlan</mark> comme autorité pour la zone <mark style="background-color:var(--base-3)" class="has-inline-color has-global-color-8-color">intra.loupipfire.fr</mark></td></tr><tr><td>1</td><td class="has-text-align-left" data-align="left"><mark style="background-color:var(--base-3)" class="has-inline-color has-global-color-8-color">N° de série</mark> du fichier, à changer à chaque modification de celui-ci</td></tr><tr><td>604800</td><td class="has-text-align-left" data-align="left">Un <mark style="background-color:var(--base-3)" class="has-inline-color has-global-color-8-color">DNS esclave</mark> attendra 1w avant de rafraichir ses données</td></tr><tr><td>84600</td><td class="has-text-align-left" data-align="left">Si échec, il réessaiera 1d après</td></tr><tr><td>2419200</td><td class="has-text-align-left" data-align="left">Si échec durant 4w, il cessera de répondre en tant qu'autorité</td></tr><tr><td>604800</td><td class="has-text-align-left" data-align="left">Mise en cache des <mark style="background-color:var(--base-3)" class="has-inline-color has-global-color-8-color">réponses négatives</mark> durant 1w</td></tr></tbody></table>

Le n° de série peut être une date suivie d'un n° d'ordre :  
Ex : AAAAMMJJxx

b) Enregistrements de type NS et A :

<table class="has-fixed-layout"><tbody><tr><td>IN NS</td><td>Indique le nom du serveur DNS pour la zone</td></tr><tr><td>IN A</td><td>Relie un nom d’hôte <em>(domaine - s/domaine)</em>&nbsp;à une adresse IPv4</td></tr></tbody></table>

Créez maintenant le fichier pour la zone inverse :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo nano db.intra.loupipfire.fr.inverse

et insérez les lignes suivantes :

;
;  DNS - Fichier de zone pour la résolution inverse
;
$TTL 86400
@ IN SOA srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. (
1
1w
1d
4w
1w )
;
@ IN NS srvlan.intra.loupipfire.fr.
1 IN PTR srvlan.intra.loupipfire.fr.
15 IN PTR ovs.intra.loupipfire.fr. 
2 IN PTR debian10-vm1.intra.loupipfire.fr. 
4 IN PTR debian10-vm2.intra.loupipfire.fr.  

\- Détail des paramètres - 
a) Directives SOA _(Start of Authority)_ :  
Idem fichier de zone pour la résolution directe.

b) Enregistrements de type PTR :

<table class="has-fixed-layout"><tbody><tr><td>IN PTR</td><td>Relie une adresse IP à un nom d'hôte</td></tr></tbody></table>

#### _3.3 - Prise en compte des modifications_

Redémarrez le service DNS :

\[srvlan@srvlan:~$\] sudo systemctl restart bind9

### 4 - Configuration finale et tests du DNS statique

#### _4.1 - Configuration du fichier hosts_

Editez le fichier DNS hosts :

\[srvlan@srvlan:~$\] sudo nano /etc/hosts

et ajoutez le FQDN de srvlan _(nom d'hôte + nom de domaine)_ comme suit :

127.0.0.1        localhost
127.0.1.1        srvlan
192.168.3.1    srvlan.intra.loupipfire.fr srvlan

Ce fichier est consulté avant l'accès au serveur bind et peut donc être utilisé pour un minimum de résolution de noms en cas de panne du serveur DNS.  
  
N'effacez pas le groupe de lignes dédié à l'IPv6.

#### _4.2 - Configuration du fichier resolv.conf_

Ce fichier indique entre autres à srvlan quels serveurs DNS interroger.  
  
Actuellement, aucun programme comme un client dhcp, network-manager ou resolvconf ne le modifie dynamiquement. Il faut donc le faire manuellement.

Editez le fichier resolv.conf :

\[srvlan@srvlan:~$\] sudo nano /etc/resolv.conf

et remplacez tout le contenu existant par ceci :

\# Fichier resolv.conf - Client DNS

# Nom du domaine local
domain intra.loupipfire.fr 

# Ajout auto du nom de domaine local aux noms d'hôtes
# non pleinement qualifiés
search intra.loupipfire.fr

# Adresses IP du ou des serveurs DNS à interroger
nameserver 192.168.3.1

#### _4.3 - Tests DNS depuis la VM srvlan_

a) Vérifiez la syntaxe du fichier /etc/bind/named.conf :

\[srvlan@srvlan:~$\] sudo named-checkconf

Sont inclus dans la vérification les fichiers DNS :  
\- named.conf.options  
\- named.conf.local  
\- named.conf.default-zones  
Aucune erreur ne sera retournée en cas de résultat OK.

b) Vérifiez la validité du fichier de zone directe :

\[srvlan@srvlan:~$\] cd /etc/bind
\[srvlan@srvlan:~$\] sudo named-checkzone -d intra.loupipfire.fr db.intra.loupipfire.fr.directe

La Cde retourne normalement :

```
loading "intra.loupipfire.fr" from "db.intra.loupipfire.fr.directe" class "IN"
zone intra.loupipfire.fr/IN: loaded serial 1
OK
```

c) Vérifiez la validité du fichier de zone inverse :

\[srlan@srvlan:~$\] sudo named-checkzone -d 3.168.192.in-addr.arpa  db.intra.loupipfire.fr.inverse  

La Cde retourne normalement :

```
loading "3.168.192.in-addr.arpa" from "db.intra.loupipfire.fr.inverse" class "IN"
zone 3.168.192.in-addr.arpa/IN: loaded serial 1
OK
```

d) Lancez les outils de vérification host et dig.  
Ceux-ci peuvent aider à détecter des problèmes dans la résolution de noms.

La Cde host est incluse dans le paquet bind9-host installé de base :

\[srvlan@srvlan:~$\] sudo host -v srvlan.intra.loupipfire.fr

Celle-ci retourne normalement :

```
Trying "srvlan.intra.loupipfire.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51244
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;srvlan.intra.loupipfire.fr.    IN  A

; ANSWER SECTION:
srvlan.intra.loupipfire.fr. 86400 IN    A   192.168.3.1

;; AUTHORITY SECTION:
intra.loupipfire.fr.    86400   IN  NS  srvlan.intra.loupipfire.fr.
 
Received 74 bytes from 127.0.0.1#53 in 2 ms
Trying "srvlan.intra.loupipfire.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10937
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0
 
;; QUESTION SECTION:
;srvlan.intra.loupipfire.fr.    IN  AAAA
 
;; AUTHORITY SECTION:
intra.loupipfire.fr.    86400   IN  SOA srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. 1 604800 84600 2419200 604800
 
Received 85 bytes from 127.0.0.1#53 in 2 ms
Trying "srvlan.intra.loupipfire.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56000
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;srvlan.intra.loupipfire.fr.    IN  MX
 
;; AUTHORITY SECTION:
intra.loupipfire.fr.    86400   IN  SOA srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. 1 604800 84600 2419200 604800
 
Received 85 bytes from 127.0.0.1#53 in 2 ms
```

La Cde dig nécessite au préalable l'installation du paquet dnsutils :

\[srvlan@srvlan:~$\] sudo apt install dnsutils
\[srvlan@srvlan:~$\] sudo dig SOA srvlan.intra.loupipfire.fr

Celle-ci retourne normalement :

```
; <<>> DiG 9.11.5-P4-5.1-Debian <<>> SOA srvlan.intra.loupipfire.fr
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7715
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
 
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: e921b304fc93f8e3650c9dca5d60fb7284ddbf9b4f7af6da (good)
;; QUESTION SECTION:
;srvlan.intra.loupipfire.fr.    IN  SOA

;; AUTHORITY SECTION:
intra.loupipfire.fr.    86400   IN  SOA srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. 1 604800 84600 2419200 604800
 
;; Query time: 7 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: sam. août 24 10:55:14 CEST 2019
;; MSG SIZE  rcvd: 124
```

e) Pour finir, testez la résolution DNS interne et externe :

\[srvlan@srvlan:~$\] ping debian10-vm1
\[srvlan@srvlan:~$\] ping debian10-vm2.intra.loupipfire.fr
\[srvlan@srvlan:~$\] ping srvlan
\[srvlan@srvlan:~$\] ping srvlan.intra.loupipfire.fr
\[srvlan@srvlan:~$\] ping mappy.fr

Le ping srvlan renvoie 127.0.0.1. 
Le ping srvlan.intra.loupipfire.fr renvoie 192.168.3.1.

### 5 - Déclaration d'un serveur DNS externe

Le DNS utilise 13 destinations de serveurs racines pour gérer les requêtes d'accès à Internet. Ces serveurs sont chargés de renvoyer lesdites requêtes vers les serveurs DNS de premier niveau appropriés _(.com, .fr, etc...)_.

Le résolveur bind9 exploite par défaut le fichier /usr/share/dns/root.hints pour interroger ces serveurs racines _(adresses de a.root-servers.net à m.root-servers.net)_ et obtenir les réponses aux requêtes d'accès à Internet.

Actuellement bind9 traite les requêtes locales _(Ex : ping debian10-vm1.intra.loupipfire.fr)_ ainsi que les requêtes d'accès à Internet _(Ex : ping www.yahoo.fr)_ et construit son cache en fonction.

Test d'une requête locale :

 \[srvlan@srvlan:~$\] nslookup ovs.intra.loupipfire.fr

Retour :

```
Server:        127.0.0.1
Address:    127.0.0.1#53

Name:    ovs.intra.loupipfire.fr
Address: 192.168.3.15
```

La VM srvlan est l'autorité pour la réponse.

Test d'une requête d'accès à Internet :

 \[srvlan@srvlan:~$\] nslookup mappy.fr

Retour :

```
Server:        127.0.0.1
Address:    127.0.0.1#53

Non-authoritative answer:
Name:    mappy.fr
Address: 193.203.32.57
```

Constat, srvlan n'est plus l'autorité pour la réponse.

Vous allez à présent, pour alléger le travail de srvlan, demander que les requêtes d'accès à Internet _(les demandes de résolutions externes)_ soient dorénavant traitées par le serveur DNS de votre box Internet.

Editez pour cela le fichier DNS named.conf.options :

\[srvlan@srvlan:~$\] sudo nano /etc/bind/named.conf.options

et modifiez celui-ci afin qu'il contienne ces lignes :

options {
     directory "/var/cache/bind";
     
     dnssec-validation auto;      \# Certaines Box gèrent mal la
                                                    \# valeur auto qu'il faut dans
                                                    \# ce cas remplacer par no 

     // Limiter les réponses récursives
     // aux réseaux/IP des interfaces du serveur DNS
     allow-recursion { localnets; }; 

     // Pour les autres réseaux/IP,
     // transfert des requêtes DNS vers la box Internet
     forward only;
     forwarders { 192.168.x.z; };      \# IP de la box Internet
     
     listen-on-v6 { any; };
};    

Redémarrez maintenant le service bind9 :

\[srvlan@srvlan:~$\] sudo systemctl restart bind9 

et testez une requête d'accès à Internet :

\[srvlan@srvlan:~$\] nslookup yahoo.fr

Résultat :

```
Server:        127.0.0.1
Address:    127.0.0.1#53

Non-authoritative answer:
Name:	yahoo.fr
Address: 124.108.115.101
Name:	yahoo.fr
Address: 212.82.100.151
Name:	yahoo.fr
Address: 106.10.248.151
Name:	yahoo.fr
Address: 98.136.103.24
Name:	yahoo.fr
Address: 74.6.136.151
```

La résolution externe se fait toujours mais utilisez-vous le DNS de la Box ?  
  
Pour le savoir, remplacez l'IP du paramètre forwarders par une IP quelconque, redémarrez le service bind9 afin de vider le cache DNS actif et relancez la requête :

\[srvlan@srvlan:~$\] nslookup yahoo.fr 

Résultat :

```
;; Got SERVFAIL reply from 127.0.0.1, trying nex...
Server:		::1
Address:	::1#53

** server can't find yahoo.fr: SERVFAIL
```

La résolution externe ne fonctionne plus, vous utilisiez bien le DNS de la Box.

Remettez maintenant la bonne adresse IP.

### 6 - Tests DNS depuis les VM de la zone LAN

#### _6.1 - VM ovs_

Editez le fichier DNS resolv.conf :

\[switch@ovs:~$\] sudo nano /etc/resolv.conf

et remplacez tout le contenu existant par ceci :

\# Fichier resolv.conf - Client DNS

# Nom du domaine local
domain intra.loupipfire.fr 

# Ajout auto du nom de domaine local aux noms d'hôtes
# non pleinement qualifiés
search intra.loupipfire.fr

# Adresses IP du ou des serveurs DNS à interroger
nameserver 192.168.3.1

Installez ensuite le paquet incluant la Cde DNS dig :

\[switch@ovs:~$\] sudo apt install dnsutils 

et vérifiez l'ID du Name Server pour intra.loupipfire.fr :

\[switch@ovs:~$\] dig NS intra.loupipfire.fr +short 

Retour :

```
srvlan.intra.loupipfire.fr.
```

Pinguez pour terminer les VM suivantes :

\[switch@ovs:~$\] ping srvlan
\[switch@ovs:~$\] ping debian10-vm1.intra.loupipfire.fr
\[switch@ovs:~$\] ping debian10-vm2

et observez les FQDN _(hôte+domaine)_ retournés.

#### _6.2 - VM debian10-vm1 et debian10-vm2_

Faites un clic droit sur l'applet réseau NetworkManager :  
\> Modifier les connexions...

Une fenêtre s'ouvre :  
\> Sélectionnez la connexion réseau affichée  
\> Cliquez sur la roue dentée située en bas à gauche

Une fenêtre s'ouvre :  
\> Onglet Paramètres IPv4  
  
Modifiez ensuite la valeur des champs suivants :  
\> Serveurs DNS > 192.168.3.1  
\> Domaines de recherche > intra.loupipfire.fr  
\> Enregistrer  
\> Authentifiez-vous si une demande est affichée

Puis redémarrez le réseau pour une MAJ des nouveaux paramètres nameserver et search :

\[client-linux@debian10-vmx:~$\] su root
Mot de passe : Entrez le MDP de l'administrateur root 
\[root@debian10-vmx:~#\] systemctl restart NetworkManager
\[root@debian10-vmx:~#\] sudo cat /etc/resolv.conf

Installez le paquet incluant la Cde DNS nslookup :

 \[root@debian10-vmx:~#\] apt install dnsutils  

et vérifiez l'IP du Name Server pour intra.loupipfire.fr :

\[root@debian10-vmx:~#\] nslookup ovs.intra.loupipfire.fr

Retour :

```
Server:		192.168.3.1
Address:	192.168.3.1#53

Name:	ovs.intra.loupipfire.fr
Address: 192.168.3.15
```

Pinguez les VM suivantes :

\[root@debian10-vmx:~#\] ping srvlan.intra.loupipfire.fr
\[root@debian10-vmx:~#\] ping ovs

et observez les FQDN _(hôte+domaine)_ retournés.

Pour terminer, lancez le navigateur Web et vérifiez le bon accès à Internet.

### 7 - Simulation de panne

Arrêtez le service DNS fourni par srvlan :

\[srvlan@srvlan:~$\] sudo systemctl stop bind9

Redémarrez les clients debian10-vm\* et ovs afin de vider leurs caches DNS.

Les pings depuis les VM debian10-vm\* du [§ 6.2](/dns-statique-debian10/#62_-_VM_debian10-vm1_et_debian10-vm2) devraient échouer.

Redémarrez le service DNS et vérifiez que les pings fonctionnent à nouveau.

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Voilà, c'est fini pour le DNS statique.  
Le mémento 7.2 vous attend à  
présent pour installer un serveur  
DHCP.

[Mémento 7.2](/dhcp-isc-dhcp-server/)
