---
title: "Pare-feu - OPNsense"
summary: Configuration d'OPNsense
authors: 
  - G.Leloup
date: 2025-08-14
categories: 
  - Pare-feu
---

![Logo OPNsense](../images/2026/01/logo-opnsense.jpg)

## Situation de base

> Le pare-feu Opnsense est installé sur un routeur comprenant 6 ports Ethernet.

Le routeur est raccordé côté WAN sur un port LAN de la box Internet. Pas de création d'une zone DMZ sur la box Internet pour l'instant car le double NAT ne semble pas gêner l'exploitation du réseau.

Un PC Debian est raccordé côté LAN du pare-feu.

Lors de l'installation d'OPNsense, les interfaces réseau du pare-feu ont été configurées comme suit :  
\- igc0 = côté WAN, igc1 = côté LAN  
\- IP WAN = 192.168.7.10 _(IP fixe de la box Internet)_  
\- IPs réseau LAN = 192.168.64.0/24  
\- Une plage d'adresses IP côté LAN a été définie.

L'accès à l'interface web d'OPNsense depuis le PC situé côté LAN fonctionne.

### Configuration d'OPNsense

Premières modifications réalisées depuis l'interface web :

#### Paramètres généraux

-> Système -> Paramètres -> Général  
\- Nom d'hôte : pareloup  
\- Domaine : parefeu.blabla.org

L'accès Internet sur OPNsense se fera depuis l'URL :  
`https://pareloup.parefeu.blabla.org:3427`

\- Fuseau horaire : Europe/Paris  
\- Langue : Français  
\- Serveurs DNS : IP de la box Internet  
\- Options du serveur DNS : Coché par défaut

#### Paramètres de l'interface web

-> Système -> Paramètres -> Administration  
\- Protocole : Mis sur HTTP au lieu de HTTPS  
\- Port TCP : Mis sur 4225 au lieu de 80 par défaut

Les valeurs des autres paramètres sont laissées par défaut.

#### Accès utilisateurs

-> Système -> Accès -> Utilisateurs  
\- Changement du MPD de l'utilisateur _root_.  
\- Création d'un nouvel utilisateur dans le groupe _admins_.  
\- Connexion avec le nouvel utilisateur et désactivation de _root_.

#### Accès web depuis Internet

<!-- more -->

-> Pare-feu -> NAT -> Redirection de port -> Icône +  
\- Interface : WAN  
\- Source : Hôte unique ou réseau = 192.168.7.0/24  
\- Plage de ports sources : any  
\- Destination : WAN adresse  
\- Plage de ports de destination : 4225  
\- Rediriger l'IP de destination : Hôte ... = 192.168.64.1

  192.168.64.1 est l'IP LAN d'OPNsense.

\- Rediriger le port cible : 4225  
\- Description : NAT HTTP OPNsense via WAN  
\- Association de règle de filtrage : Rule _Description_

-> Pare-feu -> Paramètres -> Avancé  
\- Désactiver reply-to : Cocher la case de désactivation

-> Interfaces -> WAN  
\- Bloquer les réseaux privés : Décocher la case

 WAN autorisera les URL de type `http://192.168.7.x`

\- Type de configuration IPV4 : DHCP

Si _Adresse IPv4 statique_, les PC côté LAN n'accèdent plus à Internet ?

Après application des changements, l'accès à l'interface web depuis `https://pareloup.parefeu.blabla.org:3427` fonctionne.

L'URL est préalablement aiguillée sur un reverse proxy qui renvoie la demande vers l'URL `http:192.168.7.10:4225`.

#### Serveur DHCP

Serveur Dnsmasq DNS & DHCP utilisé par défaut.

-> Services -> Dnsmasq DNS & DHCP -> Général  
\- Do not forward to system defined DNS servers : Activer  
\- DHCP FQDN : Activer  
\- DHCP default domain : Entrer maison  
\- DHCP register firewall rules : Activer

-> Services -> Dnsmasq DNS & DHCP -> DHCP ranges  
\- DHCP default domain : Entrer loup.maison

La plage d'adresses IP côté LAN apparait ici.

Entrer home dans le champ Domaine afin que les PC locaux recoivent un search de valeur home.

-> Services -> Dnsmasq DNS & DHCP -> Baux  
Le bail du PC raccordé sur igc1 apparait ici.

Une icône + permet d'accéder à la réservation d'une adresse IP fixe.

Une fois la réservation appliquée, celle-ci apparaît dans :  
-> Services -> Dnsmasq DNS et DHCP -> Hôtes

Cdes pouvant être utilisées pour renouveler le bail DHCP :  
\- Sous Linux = dhclient -r  
\- Sous Windows = ipconfig / renew

#### Serveur DNS

Serveur Dnsmasq DNS & DHCP utilisé par défaut ainsi que le serveur Unbound DNS.

Configuration Dnsmasq  
-> Services -> Dnsmasq DNS & DHCP -> Général  
\- Port d'écoute : 53317 _(port 53 utilisé par Unbound)_  

Configuration Unbound  
-> Services -> Unbound DNS -> Général  
\- Port d'écoute : 53  
\- Interface réseau : LAN au lieu de Tout par défaut  
\- Contournement de Domaine DHCP : maison au lieu de home

-> Services -> Unbound DNS -> Transmission des requêtes  
\- Domaine : loup.maison  
\- IP du serveur : 127.0.0.1  
\- IP du serveur : 53317

-> Services -> Unbound DNS -> Transmission des requêtes  
\- Domaine : 64.168.192.in-addr.arpa  
\- IP du serveur : 127.0.0.1  
\- IP du serveur : 53317

Penser à mettre les PC situés côté LAN en client DHCP afin qu'il soit traités correctement par le serveur DNS.
On a tendance à gérer une IP fixe sur les PC côté LAN, en fait il faut privilégier client DHCP et gérer ensuite un bail statique depuis le service DHCP du routeur OPNsense.

#### Protocole ICMP _(Ping)_

Le Ping implique la création d'une règle.

-> Pare-feu -> Règles -> WAN -> Icône +  
\- Protocole : ICMP  
\- Type ICMP : Demande d'écho  
\- Source : WAN net = 192.168.7.0/24  
\- Destination : any  
\- Description : Ping autorisé sur IP WAN et IPs LAN

#### Protocole SSH

-> Système -> Paramètres -> Administration  
\- Serveur Shell sécurisé : Cocher l'activation  
\- Méthode d'authentification : Cocher Autoriser avec MDP  
\- Port SSH : Par exemple 232

> Si l'utilisateur _root_ est désactivé, impossible de se connecter en SSH même si _root_ est autorisé dans la configuration Shell Secure d'OPNsense.

Pour un utilisateur normal, mettre son _Shell de connexion_ sur _/bin/sh_ et ajouter l'utilisateur dans le groupe _sudo_.

Ensuite création d'une règle de Pare-feu :  
-> Pare-feu -> Règles -> WAN -> Icône +  
\- Protocole : TCP  
\- Source : Hôte unique ou réseau = 192.168.7.x/24  
\- Destination : LAN net  
\- Plage de ports de destination : 232  
\- Description : SSH vers LAN autorisé

> Voir plus tard les connexions SSH avec clés publique/privée et non login/mdp.

#### Protocole RDP

Règle de Pare-feu :  
-> Pare-feu -> Règles -> WAN -> Icône +  
\- Protocole : TCP  
\- Source : Hôte unique ou réseau = 192.168.7.x/24  
\- Destination : LAN net  
\- Plage de ports de destination : MS RDP = port 3389  
\- Description : RDP vers LAN autorisé

#### Protocole VNC

Le MDP VNC des serveurs = vnc suivi du mdp utilisateur.

Règle de Pare-feu :  
-> Pare-feu -> Règles -> WAN -> Icône +  
\- Protocole : TCP  
\- Source : Hôte unique ou réseau = 192.168.7.x/24  
\- Destination : LAN net  
\- Plage de ports de destination : 5900 à 5910  
\- Description : VNC vers LAN autorisé

> Voir connexion VNC au travers d'un tunnel SSH.

#### Protocole HTTP

Règle de Pare-feu :  
-> Pare-feu -> Règles -> WAN -> Icône +  
\- Protocole : TCP  
\- Source : Hôte unique ou réseau = 192.168.7.x/24  
\- Destination : LAN net  
\- Plage de ports de destination : HTTP = port 80  
\- Description : HTTP vers LAN autorisé

#### Protocole SAMBA

Alias de ports _ports_samba_ créé incluant les ports 139 et 445.

Alias d'hôtes _groupe_samba_ créé incluant les IP des PC partageant des dossiers.

Règle NAT de Pare-feu :  
-> Pare-feu > NAT > Redirection de ports > Icône +  
\- Protocole : TCP  
\- Source : Alias pc_portable_utilisateur  
\- Destination : WAN adresse  
\- Plage de ports de destination : Alias ports_samba  
\- Rediriger l'IP de destination : Alias groupe_samba  
\- Rediriger le port cible : ports_samba  
\- Description : NAT SAMBA

#### Bridge pour les PC Proxmox

Au prélable, mettre le vmbr0 des PC Proxmox sur une adresse IP compatible avec le réseau LAN d'OPNsense.

Les interfaces réseau igc0 et igc1 sont déjà occupées sur le pare-feu, le PC Proxmox sera connecté sur igc2.

Au préalable, préparer le pare-feu comme suit :  
-> Pare-feu -> Interfaces -> Assignations  
Assigner igc2 et igc3 _(identifiants opt1 et opt2 créés)_

-> Pare-feu -> Interfaces -> Périphériques  
-> Bridge -> Icône +  
\- Interfaces membres : opt1, opt2

-> Pare-feu -> Interfaces -> OPT1  
Cocher Activer l'interface _(rien d'autre, idem pour OPT2)_

-> Pare-feu -> Interfaces -> Assignations  
\- Interface LAN : mettre bridge0 à la place de igc1.

Débrancher le câble LAN _(igc1)_ et le mettre sur OPT1 _(igc2)_, l'accès Web doit continuer de fonctionner.

-> Pare-feu -> Interfaces -> Assignations  
Réajouter le port igc1.

-> Pare-feu -> Interfaces -> Périphériques -> Bridge > +  
\- Interfaces membres : opt3  
\- Link-local address : Cocher Enable link-local address

-> Pare-feu -> Interfaces -> Assignations  
Les assignations peuvent être modifiées pour par exemple obtenir icgc1 = opt1, igc2 = opt2, igc3 = opt3.

-> Système -> Paramètres -> Optimisations  
\- net.link.bridge.pfil_member : mettre à 1 si nécessaire.

#### Ajout du proxmox

Connecter le Proxmox sur l'interface réseau igc2.

Depuis l'interface web de Proxmox, mettre l'IP du serveur DNS de Proxmox sur l'IP LAN d'OPNsense soit 192.168.64.1.

Faire de même pour la VM Proxmox _Nginx Proxy Manager_.

#### Règles côté LAN du pare-feu

PING et autres protocoles

Règles pour OPT2 et OPT3 créées :  
\- Source : LAN net  
\- Destination : any  
\- Plage de ports de destination : any  
\- Description : Aucun blocage côté LAN

#### Synology

Route statique créée vers le réseau IP LAN du Pare-feu.

#### VM EVE-NG

Sur le PC Windows situé côté WAN, il est nécessaire de créer en plus de la route statique menant au réseau IP LAN du Pare-feu une route donnant accès à l'interface pnet0 d'EVE-NG.

Exemple de routes :

```markdown
192.168.64.0    255.255.255.0    192.168.7.10 (LAN)  
10.128.128.0    255.255.255.0    192.168.7.10 (pnet0)
```

Ensuite sur OPNsense, créer une passerelle comme suit :

-> Système -> Passerelles -> Configuration -> Icône +  
\- Nom : Passerelle_pnet0_eve  
\- Interface : LAN  
\- Adresse IP : IP de la VM EVE-NG

Puis créer une route statique comme suit :

-> Système -> Routes -> Configuration -> Icône +  
\- Adresse Réseau : Réseau-IP-pnet/24  
\- Passerelle : Passerelle_pnet0_eve - IP VM EVE-NG

#### Alias

Penser à gérer les alias d'hôtes et de ports, ceux-ci permettant notamment d'inclure plusieurs numéros de port _(alias de ports)_ ou plusieurs adresses IP _(alias d'hôtes)_ dans un même alias.

Cela contribue à dimininuer le nombre de règles de pare-feu à écrire.

#### Erreurs corrigées

Duplication d'une route IPv6 :  
-> Interfaces -> WAN  
\- Type de configuration IPv6 : Aucun au lieu de DHCP ?

Duplication IP serveur DNS :
-> Système -> Paramètres -> Général  
\- Serveur DNS : Ne rien rentrer ?

GeoIP (pourtant non utilisé)  
Créer les dossiers suivants :  

```bash
mkdir -p /usr/local/share/GeoIP
mkdir -p /usr/local/share/GeoIP/alias
```

#### Cdes FreeBSD

Cdes diverses une fois connecté en SSH sur OpnSense.  
Passer en tant que root avec sudo -i _(même MDP que l'utilisateur principal)_.

Température zone CPU :

```bash
sysctl -a | grep -i temperature
```

Retour :

```markdown
hw.acpi.thermal.tz0.temperature: 53.1C
```

ou

```bash
kldload coretemp
sysctl dev.cpu | grep temperature
```

Pour lire le modèle de processeur :

```bash
sysctl hw.model
```

Retour :

```markdown
hw.model: Intel(R) Celeron(R) J4125 CPU @ 2.00GHz
```

Le programme top est installé par défaut, pratique pour vrifier la charge CPU.

Pour le disque SSD, installer le plugin os-smart _(smartmontools)_ depuis l'interface web Système -> Micrologiciel -> Greffons

Ensuite entrer la Cde suivante :

```bash
smartctl -a /dev/ada0  | grep Temperature
```

Retour :

```markdown
194 Temperature_Celsius     0x0023   043   030   000    Pre-fail  Always       -       57 (Min/Max 33/70)
```

Soit une température de 57° centigrade.

#### Sauvegarde de la configuration

-> Système > Configuration > Sauvegardes  
\- Téléchargement : Ne pas sauvegarder les données RRD

Penser à chiffrer le fichier de configuration avec un MDP.

**Fin.**
