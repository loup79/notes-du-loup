---
title: "Windows - Lignes de commandes"
date: "2021-04-15"
categories: 
  - "cdes-windows"
---

## Aide mémoire des Cdes DOS

### 1 - Gestion des paramètres courants du PC

_1.1 - Afficher le nom d'hôte_

\[C:\\>\] hostname

### 2 - Gestion du réseau

_2.1 - Afficher la configuration des interfaces réseau_

\[C:\\>\] ipconfig /all

_2.2 - Tracer la route empruntée par un paquet IP_

\[C:\\>\] tracert 172.98.20.3                                 \# IP de destination

\[C:\\Users\\...>\] tracert velocycle.fr      \# Domaine de destination

_2.3 - Tester une liaison réseau_

\[C:\\>\] ping 192.168.6.32                   \# OK= zéro paquets perdus

_2.4 - Lire le contenu de la table de routage_

\[C:\\>\] route print

_2.5 - Créer une route statique_

\[C:\\>\] route add 192.168.8.0 mask 255.255.255.0 192.168.x.w

Le réseau 192.168.8.0 peut ainsi être atteint en utilisant la passerelle 192.168.x.w.

La route statique sera supprimée à l'arrêt du système.

Pour la déclarer permanente, ajouter \-p dans la Cde.

_2.6 - Accéder au dossier partagé d'un serveur NAS_

\[C:\\>\] net use \\\\nom\_srv\\dossier /USER:nom\_user passwd
La commande s’est terminée correctement.

Le résultat est visible en lançant la Cde net use seule.

\---------- Fin ----------
