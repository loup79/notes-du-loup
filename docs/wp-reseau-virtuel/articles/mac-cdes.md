---
title: "OS X - Lignes de commandes"
date: "2021-04-15"
categories: 
  - "cdes-mac"
---

## Aide mémoire des Cdes OS X

Cette rubrique existe pour ceux qui utiliseraient un PC hôte supportant un système OS X d'Apple.

### 1 - Gestion du réseau

_1.1 - Créer une route statique_

\[sh-3.2#\] route add -net 192.168.6.0 -netmask 255.255.255.0 192.168.x.w

Le réseau 192.168.6.0 peut ainsi être atteint en utilisant la passerelle 192.168.x.w.

La Cde est temporaire, la route statique sera supprimée à l'arrêt du système. Il faut, pour la rendre permanente, exécuter un script à chaque démarrage du PC.

Créez le script de nom route\_statique :

\[sh-3.2#\] cd /Users/nom-utilisateur/Documents
\[sh-3.2#\] nano route\_statique

et entrez les lignes suivantes :

#!/bin/bash 
# Création d'une route statique vers le réseau virtuel

sudo /sbin/route add -net 192.168.6.0 -netmask 255.255.255.0 192.168.x.w

Rendez celui-ci exécutable :

\[sh-3.2#\] cd /Users/nom-utilisateur/Documents
\[sh-3.2#\] chmod a+x route\_statique

et forcez son lancement à chaque boot du système :

\[sh-3.2#\] sudo defaults write com.apple.loginwindows \\
LoginHook /Users/nom-utilisateur/Documents/route\_statique 

Le caractère \\ précise d'écrire la Cde sur une seule ligne.

Mettre delete à la place de write pour supprimer par la suite le lancement automatique du script.

_1.2 - Afficher la table de routage_

\[sh-3.2#\] netstat -nr

_1.3 - Tracer la route empruntée par un paquet IP_

\[sh-3.2#\] traceroute 193.203.31.52               \# IP de destination

\[sh-3.2#\] traceroute velocycle.fr        \# Domaine de destination

\---------- Fin ----------
