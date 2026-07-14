---
title: NAS - Qnap et Let's Encrypt
description: Problème de certificat non renouvelé.
authors: 
  - G.Leloup
date: 2026-07-13
categories: 
  - NAS
---

## Certificat non renouvelé

Il n'est plus possible, si le certificat SSL Let's Encrypt dédié à l'administration du Qnap est expiré car non renouvelé automatiquement, de se connecter depuis Internet car le Qnap est configuré pour accepter une connexion HTTPS uniquement.

La solution est de se connecter depuis un PC situé sur le même réseau local que le Qnap avec l'URL :
`https://ip-locale-qnap:port https`

Renouveler ensuite le certificat normalement à l'aide de l'application QTS SSL Certificate.

## Voir configuration future

Pour corriger le non renouvellement automatique, voir par la suite la gestion du port 80 au travers d'un reverse proxy avec des noms de domaine différents _(déjà pratiqué sur le reverse proxy du serveur Synology)_.

<!-- more -->

\- Solution 1  
Utiliser le reverse proxy du Synology pour cela.

\- Solution 2  
Utiliser le reverse proxy d'OPNsense soit HAProxy, plutôt que de dépendre du Synology.

Cela donnerait :

Internet  
   |  
OPNsense + HAProxy  
   |  
   +-- synology.ladomaine.tld --> Synology  
   |  
   +-- infoloup.myqnapcloud.com --> QNAP

Avantages :  
Un seul point d'entrée;  
Pas besoin de réveiller un NAS pour gérer l'accès;  
Gestion centralisée des certificats sur le pare-feu;  
Séparation claire entre réseau et stockage.

**Fin.**
