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
Ex : ping `www.google.fr`

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

53 = Port DNS TCP/UDP par défaut  
953 = Port des Cdes rndc pour gérer le démon named

Vérifiez l'activation de l'outil associé rndc :

```bash
[srvlan@srvlan:~$] sudo rndc status 
```

Retour :

```markdown
version: BIND 9.18...-Debian (Extended Supp...
running on localhost: Linux x86_64 6...-amd...
boot time: Thu, 11 Jan 2024 12:17:15 GMT
last configured: Thu, 11 Jan 2024 12:17:15 GMT
configuration file: /etc/bind/named.conf
CPUs found: 2
worker threads: 2
UDP listeners per interface: 2
number of zones: 102 (97 automatic)
debug level: 0
xfers running: 0
xfers deferred: 0
soa queries in progress: 0
query logging is OFF
recursive clients: 0/900/1000
tcp clients: 0/150
TCP high-water: 0
server is up and running
```

L'interface en ligne de Cde rndc permet de gérer le démon named localement ou à distance.

Pour finir, observez les fichiers de configuration DNS présents dans les dossiers suivants :

```bash
[srvlan@srvlan:~$] ls /etc/bind
[srvlan@srvlan:~$] ls /var/cache/bind    
```

### Configuration de base

#### _- Ajout de zones de recherche_

Une zone de recherche directe permet à un service DNS interrogé de renvoyer une adresse IP associée à un nom alors qu'une zone de recherche inverse fera le contraire.

Le fichier named.conf.local contient la configuration locale du serveur DNS bind9, vous y déclarerez donc les zones de recherche pour intra.loupipfire.fr.

Editez le fichier de configuration named.conf.local :

```bash
[srvlan@srvlan:~$] sudo nano /etc/bind/named.conf.local    
```

et ajoutez les lignes suivantes à la fin de celui-ci :

```bash
# Zone ou Domaine intra.loupipfire.fr
# Définition des zones de recherche directe et inverse
# Serveur DNS maître pour le domaine (type)
# Fichiers de zones associés dans /etc/bind (file)
# MAJ dynamique des fichiers de zones > none (allow-update)  

# zone de recherche directe
zone "intra.loupipfire.fr" {
type master;
file "/etc/bind/db.intra.loupipfire.fr.directe";
allow-update { none; };
};

# zone de recherche inverse
# Le réseau 192.168.3.0 aura pour adresse inverse 
# 3.168.192.in-addr.arpa 
zone "3.168.192.in-addr.arpa" {
type master;
file "/etc/bind/db.intra.loupipfire.fr.inverse";
allow-update { none; };
};    
```

#### _- Configuration des zones_

Les fichiers de zones de recherche contiendront les directives et enregistrements de ressources pour le domaine intra.loupipfire.fr.

Créez le fichier pour la zone directe :

```bash
[srvlan@srvlan:~$] cd /etc/bind
[srvlan@srvlan:~$] sudo nano db.intra.loupipfire.fr.directe    
```

et insérez les lignes suivantes :

```bash
;
; DNS - Fichier de zone pour la résolution directe
;
$TTL 86400
@   IN   SOA   srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. (
1                             ; Serial
604800                  ; Refresh - 1w (1 semaine)
84600                    ; Retry - 1d (1 jour)
2419200               ; Expire - 4w 
604800 )               ; Negative Cache TTL - 1w
;
@       IN     NS     srvlan.intra.loupipfire.fr.
srvlan       IN     A     192.168.3.1
ovs       IN     A     192.168.3.15
debian12-vm1       IN     A     192.168.3.2
debian12-vm2       IN     A     192.168.3.4
ctn1       IN     A     192.168.3.6
ctn2       IN     A     192.168.3.8    
```

\- Détail des paramètres -  
a) Directives SOA (Start of Authority) :

|                    |                                                                               |
| :----------------- | :---------------------------------------------------------------------------- |
| @ {.td-bord}       | Représente le nom de domaine de la zone soit intra.loupipfire.fr {.td-bord}   |
| IN SOA {.td-bord}  | Désigne srvlan comme autorité pour la zone intra.loupipfire.fr {.td-bord}     |
| 1 {.td-bord}       | N° de série du fichier, à changer à chaque modification de celui-ci {.td-bord}|
| 604800 {.td-bord}  | Un DNS esclave attendra 1w avant de rafraichir ses données {.td-bord}         |
| 84600 {.td-bord}   | Si échec, il réessaiera 1d après  {.td-bord}                                  |
| 2419200 {.td-bord} | Si échec durant 4w, il cessera de répondre en tant qu'autorité {.td-bord}     |
| 604800 {.td-bord}  | Mise en cache des réponses négatives durant 1w  {.td-bord}                    |

Le n° de série peut être une date suivie d'un n° d'ordre :  
Ex : AAAAMMJJxx

b) Enregistrements de type NS et A :

|                    |                                                                               |
| :----------------- | :---------------------------------------------------------------------------- |
| IN NS {.td-bord}   | Indique le nom du serveur DNS pour la zone  {.td-bord}                        |
| IN A {.td-bord}    | Relie un nom d’hôte (domaine - s/domaine) à une adresse IPv4 {.td-bord}       |

Créez maintenant le fichier pour la zone inverse :

```bash
[srvlan@srvlan:~$] cd /etc/bind
[srvlan@srvlan:~$] sudo nano db.intra.loupipfire.fr.inverse    
```

et insérez les lignes suivantes :

```bash
;
;  DNS - Fichier de zone pour la résolution inverse
;
$TTL 86400
@   IN   SOA   srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. (
1
1w
1d
4w
1w )
;
@       IN     NS     srvlan.intra.loupipfire.fr.
1       IN     PTR     srvlan.intra.loupipfire.fr.
15       IN     PTR     ovs.intra.loupipfire.fr. 
2       IN     PTR     debian12-vm1.intra.loupipfire.fr. 
4       IN     PTR     debian12-vm2.intra.loupipfire.fr.
6       IN     PTR     ctn1.intra.loupipfire.fr.
8       IN     PTR     ctn2.intra.loupipfire.fr.    
```

\- Détail des paramètres -  
a) Directives SOA (Start of Authority) :  
Idem fichier de zone pour la résolution directe.

b) Enregistrements de type PTR :

|                    |                                                                               |
| :----------------- | :---------------------------------------------------------------------------- |
| IN PTR {.td-bord}  | Relie une adresse IP à un nom d'hôte  {.td-bord}                              |

!!! note "Nota"

    Le conteneur sécurisé rootless ctn3 n'a pas été traité, le service réseau slirp4netns qui lui est associé manque de certaines fonctionnalités notamment celle de ne pas donner au conteneur une adresse IP routable.

#### _- Mise à jour de Bind 9_

Redémarrez le service DNS :

```bash
[srvlan@srvlan:~$] sudo systemctl restart bind9    
```

### Configuration finale et tests

#### _- Refonte fichier hosts_

Editez le fichier DNS hosts :

```bash
[srvlan@srvlan:~$] sudo nano /etc/hosts    
```

et ajoutez le FQDN de srvlan (==nom d'hôte + nom de domaine==) comme suit :

```bash
127.0.0.1        localhost
127.0.1.1        srvlan
192.168.3.1    srvlan.intra.loupipfire.fr srvlan   
```

Ce fichier est consulté avant l'accès au serveur bind et peut donc être utilisé pour un minimum de résolution de noms en cas de panne du serveur DNS.

N'effacez pas le groupe de lignes dédié à l'IPv6.

#### _- Refonte fichier resolv.conf_

Vous allez modifier ce fichier actuellement traité dynamiquement par l'outil NetworkManager et qui indique à srvlan quel serveur DNS interroger.

Faites un clic droit sur l'applet réseau NetworkManager :  
-> Modifier les connexions...

Une fenêtre Connexions réseau s'ouvre :  
-> Sélectionnez la Connexion carte 2  
-> Cliquez sur la roue dentée située en bas à gauche

Une fenêtre Modification de la Connexion ... s'ouvre :  
-> Onglet Paramètres IPv4

Modifiez ensuite la valeur des champs suivants :  
-> Serveurs DNS > 192.168.3.1  
-> Domaines de recherche > intra.loupipfire.fr  
-> Bouton Enregistrer  
-> Authentifiez-vous si une demande est affichée

Modifiez ensuite la Connexion carte 1 en effaçant le contenu du champ Serveurs DNS.

Puis redémarrez le réseau pour une MAJ des nouveaux paramètres nameserver et search :

```bash
[srvlan@srvlan:~$] sudo systemctl restart NetworkManager  
```

Affichez pour finir le contenu du fichier resolv.conf :

```bash
[srvlan@srvlan:~$] cat /etc/resolv.conf 
```

qui doit montrer ceci :

```markdown
# Generated by NetworkManager
search intra.loupipfire.fr
nameserver 192.168.3.1
```

#### _- Tests du DNS depuis srvlan_

a) Vérifiez la syntaxe du fichier /etc/bind/named.conf :

```bash
[srvlan@srvlan:~$] sudo named-checkconf 
```

Sont inclus dans la vérification les fichiers DNS :  
\- named.conf.options  
\- named.conf.local  
\- named.conf.default-zones  
Aucune erreur ne sera retournée en cas de résultat OK.

b) Vérifiez la validité du fichier de zone directe :

```bash
[srvlan@srvlan:~$] cd /etc/bind

[srvlan@srvlan:~$] sudo named-checkzone -d intra.loupipfire.fr db.intra.loupipfire.fr.directe 
```

La Cde retourne normalement :

```markdown
loading "intra.loupipfire.fr" from "db.intra.loupipfire.fr.directe" class "IN"
zone intra.loupipfire.fr/IN: loaded serial 1
OK 
```

c) Vérifiez la validité du fichier de zone inverse :

```bash
[srlan@srvlan:~$] sudo named-checkzone -d 3.168.192.in-addr.arpa  db.intra.loupipfire.fr.inverse   
```

La Cde retourne normalement :

```markdown
loading "3.168.192.in-addr.arpa" from "db.intra.loupipfire.fr.inverse" class "IN"
zone 3.168.192.in-addr.arpa/IN: loaded serial 1
OK
```

d) Testez les outils de vérification host et dig :  
Ceux-ci peuvent aider à détecter des problèmes dans la résolution de noms.

La Cde host est incluse dans le paquet bind9-host installé de base :

```bash
[srvlan@srvlan:~$] sudo host -v srvlan.intra.loupipfire.fr  
```

Celle-ci retourne normalement :

```markdown
Trying "srvlan.intra.loupipfire.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9400
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;srvlan.intra.loupipfire.fr. IN A

;; ANSWER SECTION:
srvlan.intra.loupipfire.fr. 86400 IN A 192.168.3.1

Received 60 bytes from 192.168.3.1#53 in 16 ms
Trying "srvlan.intra.loupipfire.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64294
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;srvlan.intra.loupipfire.fr. IN AAAA

;; AUTHORITY SECTION:
intra.loupipfire.fr. 86400 IN SOA srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. 1 604800 84600 2419200 604800

Received 85 bytes from 192.168.3.1#53 in 0 ms
Trying "srvlan.intra.loupipfire.fr"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51406
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;srvlan.intra.loupipfire.fr IN MX

;; AUTHORITY SECTION:
intra.loupipfire.fr. 86400 IN SOA srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. 1 604800 84600 2419200 604800

Received 85 bytes from 192.168.3.1#53 in 0 ms
```

La Cde dig est incluse dans le paquet bind9-dnsutils installé de base :

```bash
[srvlan@srvlan:~$] sudo dig SOA srvlan.intra.loupipfire.fr  
```

Celle-ci retourne normalement :

```markdown
; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> SOA srvlan.intra.loupipfire.fr
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26365
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ce938125a123e1bd0100000065a1... (good)
;; QUESTION SECTION:
;srvlan.intra.loupipfire.fr. IN SOA

;; AUTHORITY SECTION:
intra.loupipfire.fr. 86400 IN SOA srvlan.intra.loupipfire.fr. root.intra.loupipfire.fr. 1 604800 84600 2419200 604800

;; Query time: 0 msec
;; SERVER: 192.168.3.1#53(192.168.3.1) (UDP)
;; WHEN: Fri Jan 12 17:26:23 CET 2024
;; MSG SIZE  rcvd: 124  
```

e) Testez la résolution DNS interne et externe :

```bash
[srvlan@srvlan:~$] ping debian12-vm1
[srvlan@srvlan:~$] ping debian12-vm2.intra.loupipfire.fr
[srvlan@srvlan:~$] ping srvlan
[srvlan@srvlan:~$] ping srvlan.intra.loupipfire.fr
[srvlan@srvlan:~$] ping ctn2
[srvlan@srvlan:~$] ping lemonde.fr  
```

Le ping srvlan renvoie 127.0.1.1.  
Le ping srvlan.intra.loupipfire.fr renvoie 192.168.3.1.

f) Pour finir, vérifiez le chargement des zones :

```bash
sudo named-checkconf -z   # Liste des zones chargées 
```

### Déclaration d'un DNS externe

Le DNS utilise 13 destinations de serveurs racines pour gérer les requêtes d'accès à Internet. Ces serveurs sont chargés de renvoyer lesdites requêtes vers des serveurs DNS de premier niveau appropriés _(.com, .fr, etc...)_.

Le résolveur bind9 exploite par défaut le fichier /usr/share/dns/root.hints pour interroger ces serveurs racines _(adresses de a.root-servers.net à m.root-servers.net)_ et obtenir les réponses aux requêtes d'accès à Internet.

Actuellement bind9 traite les requêtes locales _(Ex : ping debian12-vm1.intra.loupipfire.fr)_ ainsi que les requêtes d'accès à Internet _(Ex : ping lemonde.fr)_ et construit son cache en fonction.

Test d'une requête locale :

```bash
[srvlan@srvlan:~$] nslookup ovs.intra.loupipfire.fr 
```

Retour :

```markdown
Server: 192.168.3.1 (IP serveur DNS soit srvlan)
Address:    192.168.3.1#53

Name: ovs.intra.loupipfire.fr
Address: 192.168.3.15
```

La VM srvlan est l'autorité pour la réponse.

Test d'une requête d'accès à Internet :

```bash
[srvlan@srvlan:~$] nslookup mappy.fr
```

Retour :

```markdown
Server:     192.168.3.1 (IP serveur DNS soit srvlan)
Address:    192.168.3.1#53

Non-authoritative answer:
Name:    mappy.fr
Address: 193.203.32.57
```

Constat, srvlan n'est plus l'autorité pour la réponse.

Vous allez à présent, pour alléger le travail de srvlan, demander que les requêtes d'accès à Internet _(les demandes de résolutions externes)_ soient dorénavant traitées par les serveurs DNS de Google.

Editez pour cela le fichier DNS named.conf.options :

```bash
[srvlan@srvlan:~$] sudo nano /etc/bind/named.conf.options
```

et modifiez celui-ci afin qu'il contienne ces lignes :

```bash
options {
     directory "/var/cache/bind";
     
     dnssec-validation auto;      

     // Limiter les réponses récursives aux
     // réseaux/IP des interfaces du serveur DNS srvlan
     allow-recursion { localnets; }; 

     // Pour les autres réseaux/IP,
     // transfert des requêtes DNS vers Internet.
     // Adresses IPv4 des serveurs DNS de Google.
     forward only;
     forwarders { 8.8.8.8; 8.8.4.4; }; 
     
     listen-on-v6 { any; };
};    
```

Redémarrez maintenant le service bind9 :

```bash
[srvlan@srvlan:~$] sudo systemctl restart bind9 
```

et testez une requête d'accès à Internet :

```bash
[srvlan@srvlan:~$] nslookup yahoo.fr 
```

Retour :

```markdown
Server: 192.168.3.1
Address: 192.168.3.1#53

Non-authoritative answer:
Name: yahoo.fr
Address: 44.228.206.170
Name: yahoo.fr
Address: 34.213.101.254
Name: yahoo.fr
Address: 13.50.184.192
Name: yahoo.fr
Address: 13.251.69.97
...
```

La résolution externe se fait toujours mais utilisez-vous les DNS de Google ?

Pour le savoir, remplacez les IP du paramètre forwarders par des IP quelconques, redémarrez le service bind9 afin de vider le cache DNS actif et relancez la requête :

```bash
[srvlan@srvlan:~$] nslookup yahoo.fr  
```

Retour :

```markdown
;; communications error to 192.168.3.1#53: timed out
Server: 192.168.3.1
Address: 192.168.3.1#53

** server can't find yahoo.fr: SERVFAIL 
```

La résolution externe ne fonctionne plus, vous utilisiez bien les DNS de Google.

Remettez maintenant les bonnes adresses IP er redémarrez le service bind9.

### Tests DNS depuis les VM/CTN

#### _- VM ovs et CTN ctn1/ctn2_

Se connecter en SSH sur la VM ovs _([voir Mémento 6.1](../posts/controle-distant-debian12.md){ target="_blank" })_.

Editez ensuite son fichier DNS resolv.conf :

```bash
[switch@ovs:~$] sudo nano /etc/resolv.conf  
```

et remplacez le contenu existant par celui-ci :

```bash
# Fichier resolv.conf - Client DNS

# Nom du domaine local
domain intra.loupipfire.fr 

# Ajout auto du nom de domaine local aux
# noms d'hôtes non pleinement qualifiés
search intra.loupipfire.fr

# Adresses IP du ou des serveurs DNS à interroger
nameserver 192.168.3.1  
```

Rebootez la VM :

```bash
[switch@ovs:~$] sudo reboot 
```

Puis, utilisez cette Cde pour interagir avec ctn1/ctn2 :

```bash
[switch@ovs:~$] sudo podman exec -it ctn1 ou 2 bash 
```

et contrôlez le contenu des 2 fichiers resolv.conf qui doivent montrer ceci :

```markdown
search intra.loupipfire.fr
nameserver 192.168.3.1 
```

Celui-ci a été mis à jour automatiquement lors de la reconstruction des 2 conteneurs Podman.

Vérifiez l'ID du Name Server pour intra.loupipfire.fr :

```bash
[switch@ovs:~$] dig NS intra.loupipfire.fr +short  
```

Retour :

```markdown
srvlan.intra.loupipfire.fr.  
```

Pour finir, depuis les 3 systèmes, pinguez les VM suivantes et observez les FQDN retournés :

```bash
ping srvlan
ping debian12-vm1.intra.loupipfire.fr
ping debian12-vm2  
```

FQDN = nom_hôte.nom_domaine

#### _- VM debian12-vm*_

Idem srvlan.