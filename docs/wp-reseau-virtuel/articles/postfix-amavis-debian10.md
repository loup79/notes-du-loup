---
title: "E-mail / Debian 10 : 2/3"
date: "2022-04-13"
categories: 
  - "archives"
---

## Mémento 11.2 - Antispam et Antivirus

Amavisd-New assurera la relation entre le MTA Postfix et des outils de filtrage du courrier indésirable tels l'anti-virus ClamAV et l'anti-spam SpamAssassin.

### 10 - Filtrage du courrier indésirable

#### _10.1 - Ajout de Amavisd-New_

Le filtrage du courrier consommera de la ressource mémoire. Pour pouvoir continuer, il vous faut augmenter si possible celle-ci de 384 Mo à au moins 2 Go.

A défaut de mémoire disponible, vous pouvez ajouter une mémoire virtuelle de 3 Go en créant un fichier swap qui viendra compléter la partition swap déjà existante.

Suivez [ce lien](/linux-cdes/#10_-_Gestion_dun_fichier_Swap) pour créer un fichier swap.

Une fois la mémoire étendue, installez Amavisd-New :

\[srvdmz@srvdmz:~$\] sudo apt  install amavisd-new

Une alerte concernant le nom d'hôte pleinement qualifié du serveur _(FQDN)_ apparaîtra et entrainera le non démarrage automatique du service amavis.

Pour corriger cela, éditez le fichier 05-node\_id :

\[srvdmz@srvdmz:~$\] cd /etc/amavis/conf.d
\[srvdmz@srvdmz:~$\] sudo nano 05-node\_id

Puis décommentez et modifiez la ligne commençant par $myhostname comme suit :

$myhostname = "srvdmz.loupvirtuel.fr";

Démarrez ensuite manuellement le service amavis :

\[srvdmz@srvdmz:~$\] sudo systemctl start amavis

et contrôlez son fonctionnement à l'aide de ces 2 Cdes :

\[srvdmz@srvdmz:~$\] telnet 127.0.0.1 10024 \# quit pour fermer
\[srvdmz@srvdmz:~$\] sudo lsof -i | grep 10024     \# port amavis

Un groupe/utilisateur amavis a été créé sur le système.

Relevez maintenant la liste des décodeurs ou autres déclarés comme non installés :

\[srvdmz@srvdmz:~$\]  sudo systemctl status amavis

et ajoutez ceux manquants :

\[srvdmz@srvdmz:~$\] sudo apt install rpm arj cabextract nomarch unrar-free p7zip-full lz4 lzop lrzip altermime

Redémarrez amavis et vérifiez son nouveau statut :

\[srvdmz@srvdmz:~$\] sudo systemctl restart amavis
\[srvdmz@srvdmz:~$\] sudo systemctl status amavis

[![Capture - Amavisd-New : Statut du service amavis](/wp-content/uploads/2021/04/service-amavisd-new-430x168.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/service-amavisd-new.jpg)

Amavisd-New : Statut du service amavis

Seuls les décodeurs pour les fichiers joints .doc et .zoo ne sont pas fournis par Debian 10 et le traitement de l'extension .F est désactivé par défaut.

Pour consulter les logs, utilisez les Cdes suivantes :

\[srvdmz@srvdmz:~$\] sudo  cat /var/log/syslog | grep amavis
\[srvdmz@srvdmz:~$\] sudo cat /var/log/mail.log | grep amavis

#### _10.2 - Ajout du scanner de virus ClamAV_

Installez les 2 paquets suivants :

\[srvdmz@srvdmz:~$\] sudo apt  install clamav clamav-daemon

Un groupe/utilisateur clamav et 2 services ont été créés.

Vérifiez le statut du service de MAJ des Bdd ClamAV :

\[srvdmz@srvdmz:~$\] sudo systemctl status clamav-freshclam

[![Capture - ClamAV : Statut du service clamav-freshclam](/wp-content/uploads/2021/04/service-clamav-freshclam-430x138.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/service-clamav-freshclam.jpg)

ClamAV : Statut du service clamav-freshclam

ainsi que ses logs :

\[srvdmz@srvdmz:~$\] sudo journalctl -eu clamav-freshclam

[![Capture - ClamAV : Logs du service clamav-freshclam](/wp-content/uploads/2021/04/service-clamav-freshclam-logs-430x184.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/service-clamav-freshclam-logs.jpg)

ClamAV : Logs du service clamav-freshclam

Les Bdd daily, main et bytecode doivent être updated.

Redémarrez le service clamav-freshclam :

\[srvdmz@srvdmz:~$\] sudo systemctl restart clamav-freshclam
\[srvdmz@srvdmz:~$\] sudo systemctl status clamav-freshclam

Si Bdd up to date, démarrez le service clamav-daemon :

\[srvdmz@srvdmz:~$\] sudo systemctl start clamav-daemon
\[srvdmz@srvdmz:~$\] sudo systemctl status clamav-daemon

[![Capture - ClamAV : Statut du service clamav-daemon](/wp-content/uploads/2021/04/service-clamav-daemon-430x201.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/service-clamav-daemon.jpg)

ClamAV : Statut du service clamav-daemon

Au prochain boot système, le service clamav-daemon démarrera automatiquement.

Le scan d'un dossier peut déjà s'effectuer comme suit :

\[srvdmz@srvdmz:~$\] sudo clamscan /home/srvdmz

[![Capture - ClamAV : Scan du dossier /home/srvdmz](/wp-content/uploads/2021/04/clamav-scan-dossier-373x430.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/clamav-scan-dossier.jpg)

ClamAV : Scan du dossier /home/srvdmz

Les logs sont consultables dans /var/log/clamav/.

#### _10.3 - Ajout de l'anti-spam SpamAssassin_

Installez le paquet suivant :

\[srvdmz@srvdmz:~$\] sudo apt  install spamassassin
\[srvdmz@srvdmz:~$\] sudo systemctl start spamassassin
\[srvdmz@srvdmz:~$\] sudo systemctl status spamassassin

[![Capture - SpamAssassin : Statut du service spamassassin](/wp-content/uploads/2021/04/service-spamassassin-430x223.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/service-spamassassin.jpg)

SpamAssassin : Statut du service spamassassin

Si OK, activez spamassassin au boot du système :

\[srvdmz@srvdmz:~$\] sudo systemctl enable spamassassin

Un groupe/utilisateur debian-spamd a été créé.

Pour consulter les logs, utilisez la Cde :

\[srvdmz@srvdmz:~$\] sudo cat /var/log/mail.log | grep spamd

#### _10.4 - Réglage de Postfix et Amavisd-New_

\- - Configuration Postfix - -

Amavis travaille comme un proxy SMTP. Le courrier entrant transféré par Postfix via SMTP est traité et retourné vers celui-ci via une nouvelle connexion SMTP.

Pour transférer le courrier entrant, éditez main.cf :

\[srvdmz@srvdmz:~$\]  sudo nano /etc/postfix/main.cf

et ajoutez les lignes suivantes à la fin de celui-ci :

\# Transfert du courrier entrant vers Amavis pour filtrage
content\_filter = smtp-amavis:\[127.0.0.1\]:10024
receive\_override\_options = no\_address\_mappings

La 3ème ligne désactive les réécritures d'adresses _(alias, etc... )_ avant filtrage.

Editez également le fichier master.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/master.cf

Ajoutez ceci pour la liaison SMTP Postfix vers Amavis :

\# Connexion SMTP de Postfix vers Amavis
smtp-amavis unix - - - - 2 smtp
    -o smtp\_data\_done\_timeout=1200  
    -o smtp\_send\_xforward\_command=yes  
    -o disable\_dns\_lookups=yes  
    -o max\_use=20 

et ceci pour la liaison SMTP Amavis vers Postfix :

\# Connexion SMTP de Amavis vers Postfix
127.0.0.1:10025 inet n - - - - smtpd  
    -o content\_filter=
    -o mynetworks=127.0.0.0/8
    -o local\_recipient\_maps=  
    -o relay\_recipient\_maps=  
    -o smtpd\_restriction\_classes=  
    -o smtpd\_delay\_reject=no  
    -o smtpd\_client\_restrictions=permit\_mynetworks,reject  
    -o smtpd\_helo\_restrictions=  
    -o smtpd\_sender\_restrictions=  
    -o smtpd\_recipient\_restrictions=permit\_mynetworks,reject  
    -o smtpd\_data\_restrictions=reject\_unauth\_pipelining  
    -o smtpd\_end\_of\_data\_restrictions=  
    -o smtpd\_error\_sleep\_time=0  
    -o smtpd\_soft\_error\_limit=1001  
    -o smtpd\_hard\_error\_limit=1000  
    -o smtpd\_client\_connection\_count\_limit=0  
    -o smtpd\_client\_connection\_rate\_limit=0  
    -o receive\_override\_options=no\_header\_body\_checks,\\
        no\_unknown\_recipient\_checks  
    -o smtpd\_tls\_security\_level=none

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Terminez en plaçant les 4 lignes suivantes sous celle commençant par pickup unix ... :

pickup unix   n    - - 60    1    pickup   \# Ligne existante
# Amavis début   
   -o content\_filter=   
   -o receive\_override\_options=no\_header\_body\_checks   
# Amavis fin

Cela empêchera les messages générés pour signaler du spam d'être classés comme spam.

_Nota : Toutes les lignes ci-dessus commençant par \-o doivent être indentées de 2 espaces._

Redémarrez le service postfix :

\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

et vérifiez qu'il écoute bien sur le port SMTP n° 10025 :

\[srvdmz@srvdmz:~$\] telnet 127.0.0.1 10025

Ctrl droite + AltGr + \] = ^\] => prompt telnet > quit

[![Capture - Postfix : Ecoute sur le port SMTP 10025](/wp-content/uploads/2021/04/postfix-telnet-port-10025-430x136.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/postfix-telnet-port-10025.jpg)

Postfix : Ecoute sur le port SMTP 10025

\- - Configuration Amavis - -

Pour info, les fichiers de configuration situés dans /etc/amavis/conf.d/ sont lus par ordre alphabétique. Un même paramètre présent dans ces fichiers verra la lecture de sa dernière valeur prise en compte. Le fichier 50-user est le dernier à être lu.

La détection anti-virus et anti-spam pour les courriers n'est pas activée par défaut.

Pour l'activer, éditez le fichier 15-content\_filter\_mode :

\[srvdmz@srvdmz:~$\] cd /etc/amavis/conf.d
\[srvdmz@srvdmz:~$\] sudo nano 15-content\_filter\_mode

et décommentez les 4 lignes de bypass ci-dessous :

use strict;

# You can ... re-enable SPAM checking through spamassassin
# and to re-enable antivirus checking.

#
# Default antivirus checking mode
# Please note, that anti-virus checking is DISABLED by
# default.
# If ... to enable it, please uncomment the following lines:

 @bypass\_virus\_checks\_maps = (
    \\%bypass\_virus\_checks, \\@bypass\_virus\_checks\_acl, \\$bypass\_virus\_checks\_re);

#
# Default SPAM checking mode
# Please note, that anti-spam checking is DISABLED by
# default.
# If ... to enable it, please uncomment the following lines:

 @bypass\_spam\_checks\_maps = (
    \\%bypass\_spam\_checks, \\@bypass\_spam\_checks\_acl, \\$bypass\_spam\_checks\_re);

Exécutez les Cdes suivantes afin que amavis et clamav puissent partager leurs informations :

\[srvdmz@srvdmz:~$\] sudo adduser clamav amavis
\[srvdmz@srvdmz:~$\] sudo adduser amavis clamav

Amavis communiquera avec ClamAV via le socket Unix /var/run/clamav/clamd.ctl.

Réduisez la consommation des ressources système en éditant le fichier 50-user :

\[srvdmz@srvdmz:~$\]  sudo nano /etc/amavis/conf.d/50-user

et en ajoutant ceci juste avant la ligne Do not modify ... :

$max\_servers = 2;

Amavis verra ainsi ses processus enfants limités à 2.

La Cde ci-dessous permet de connaitre les ressources CPU et mémoire consommées :

\[srvdmz@srvdmz:~$\] ps aux | grep '%MEM\\|amavis'

Redémarrez le service amavis et contrôlez son statut :

\[srvdmz@srvdmz:~$\] sudo systemctl restart amavis
\[srvdmz@srvdmz:~$\] sudo systemctl status amavis 

[![Capture - Amavisd-New : Scanners de ClamAV pris en compte](/wp-content/uploads/2021/04/service-amavisd-new-bis-430x193.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/service-amavisd-new-bis.jpg)

Amavisd-New : Scanners de ClamAV pris en compte

Le scanner utilisé soit le primary travaille en RAM, le secondary plus lent sert de secours.

#### _10.5 - Réglage du scanner de virus ClamAV_

Les fichiers sont situés dans /etc/clamav/.

Il est possible de réduire la fréquence de MAJ des Bdd en éditant le fichier freshclam.conf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/clamav/freshclam.conf

et en modifiant la valeur du paramètre Checks à 3 :

Checks 3

Les MAJ seront effectuées 3 fois par jour au lieu de 24.

Un restart manuel de clamav-daemon provoque ceci :

\[srvdmz@srvdmz:~$\] sudo systemctl restart clamav-daemon
\[srvdmz@srvdmz:~$\] sudo systemctl status clamav-daemon

.../mkdir /run/clamav (code=exited, status=1/FAILURE)

Pour corriger cela, éditez le fichier extend.conf :

\[srvdmz@srvdmz:~$\] cd /etc/systemd/system
\[srvdmz@srvdmz:~$\] sudo nano clamav-daemon.service.d/extend.conf

et modifiez la ligne contenant mkdir comme suit :

ExecStartPre=-/bin/mkdir \-p /run/clamav

Rechargez la config systemd et redémarrez le service :

\[srvdmz@srvdmz:~$\] sudo systemctl daemon-reload
\[srvdmz@srvdmz:~$\] sudo systemctl restart clamav-daemon
\[srvdmz@srvdmz:~$\] sudo systemctl status clamav-daemon

Celui-ci fonctionnera dorénavant correctement.

Si vous avez créé un fichier swap, ajoutez ces lignes dans le fichier extend.conf :

IOSchedulingPriority = 7
CPUSchedulingPolicy = 5
MemoryMax=128M
CPUQuota=10%
Nice = 19
TimeoutStartSec=8mn

Rebootez et observez avec conky le remplissage progressif de celui-ci par clamav _(~1,5 Go)_.

#### _10.6 - Réglage de l'anti-spam SpamAssassin_

Les fichiers sont situés dans /etc/spamassassin/ et /etc/default/.

Pour une meilleure détection des spams, installez les 2 plugins pyzor et razor :

\[srvdmz@srvdmz:~$\]  sudo apt install pyzor razor

Initialisez ensuite razor :

\[srvdmz@srvdmz:~$\] su root
\[root@srvdmz:~#\]  su amavis -c 'razor-admin -create'
\[root@srvdmz:~#\]  su amavis -c 'razor-admin -register'

Retour :

```
Register successful.  Identity stored in /var/lib/amavis/.razor/identity-ruA4...
```

Déclarez l'utilisation des 2 plugins en éditant local.cf :

\[root@srvdmz:~#\]  exit
\[srvdmz@srvdmz:~$\] sudo nano /etc/spamassassin/local.cf

et en ajoutant les 3 lignes suivantes à la fin du fichier :

\# Utilisation pour chaque courrier des plugins Pyzor et Razor
use\_pyzor 1
use\_razor2 1

Redémarrez le service spamassassin :

\[srvdmz@srvdmz:~$\]  sudo systemctl restart spamassassin

Pour tester l'usage des plugins, téléchargez ce fichier :

\[srvdmz@srvdmz:~$\]  cd /home/srvdmz
\[srvdmz@srvdmz:~$\] wget https://github.com/apache/\\
                      spamassassin/blob/trunk/sample-spam.txt

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Lancez ensuite la Cde ci-dessous :

\[srvdmz@srvdmz:~$\] su root
\[root@srvdmz:~#\]  su debian-spamd -c "spamassassin -D -t \\
< sample-spam.txt 2>&1 | egrep 'pyzor|razor'"

Le caractère \\ indique d'écrire le tout sur une seule ligne.

et observez ces lignes dans le retour affiché :

```
Apr..dbg: pyzor: network tests on, attempting Pyzor
Apr..dbg: razor2: razor2 is available, version 2.84
Apr..dbg: config:../var/lib/spamass.../25_pyzor.cf
Apr..dbg: config:../25_pyzor.cf" for included file
Apr..dbg: config: read file..spamass../25_pyzor.cf
Apr..dbg: config:../var/lib/spamass../25_razor2.cf
Apr..dbg: config:../25_razor2.cf" for included file
Apr..dbg: config: read file..spamas../25_razor2.cf
Apr..dbg: razor2: part=0 engine=8 contested=0 con..
Apr..dbg: razor2: results: spam? 0
Apr..dbg: razor2: results: engine 8, highest cf s..
Apr..dbg: razor2: results: engine 4, highest cf s..
Apr..dbg:..executable for pyzor was found at /us..
Apr..dbg: pyzor: pyzor is available: /usr/bin/pyzor
Apr..dbg: pyzor: opening..pyzor check < /tmp/.sp..
Apr..dbg: pyzor: [4736] finished: exit 1
Apr..dbg: pyzor:..res..pyzor.org:24441 (200, 'OK')
Apr..dbg: timing:..check_razor2:..%), check_pyzor:
```

\[root@srvdmz:~#\]  exit

Vous allez à présent mettre à jour les règles anti-spam.

Au préalable, vérifiez que gnupg est installé sinon :

\[srvdmz@srvdmz:~$\]  sudo apt install gnupg

Listez les modules perl requis mais vus manquants :

\[srvdmz@srvdmz:~$\] spamassassin -D --lint 2>&1 | grep -i failed

et installez ceux concernés :

\[srvdmz@srvdmz:~$\]  sudo apt install libencode-detect-perl \\
libgeo-ip-perl \\
libmail-spf-perl \\
libnet-cidr-lite-perl \\
liblwp-useragent-determined-perl \\
libnet-patricia-perl \\
libbsd-resource-perl

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Pour le module perl digest::SHA1, vérifiez que les paquets suivants sont installés sur srvdmz :

make, gcc et libc6-dev

et installez le module comme ceci :

\[srvdmz@srvdmz:~$\] sudo cpan install Digest::SHA1

Retour :

```
Would you ... configure ... automati...? [yes] Enter
```

L'installation du module doit se terminer normalement.

Mettez enfin à jour les règles anti-spam :

\[srvdmz@srvdmz:~$\]  sudo sa-update -D

La MAJ est observable dans le dossier suivant :

\[srvdmz@srvdmz:~$\] cd /var/lib/spamassassin/3.004002
\[srvdmz@srvdmz:~$\] sudo ls -l updates\_spamassassin\_org

Le service spamassassin assurera par la suite cette mise à jour automatiquement.

### 11 - Tests

#### _11.1 - Détection automatique d'un SPAM_

Par défaut, les courriers détectés comme SPAM sont mis en quarantaine dans un sous-dossier de /var/lib/amavis/virusmails/ et transmis aux destinataires marqués du tag \*\*\*SPAM\*\*\* dans le champ Sujet.

Le fichier /etc/amavis/conf.d/20-debian\_defaults montre les paramètres suivants :

$sa\_spam\_subject\_tag = '\*\*\*SPAM\*\*\*  ';                       \# TAG
$final\_spam\_destiny       = D\_PASS;            \# SPAM transmis

Envoyez, depuis srvlan, un courrier de postmaster vers clientmail-vm2 contenant ceci :

Début
XJS\*C4JDBQADN1.NSBN3\*2IDNEN\*GTUBE-STANDARD-\\
ANTI-UBE-TEST-EMAIL\*C.34X
Ce courrier doit normalement être traité comme spam.
Le destinataire clientmail-vm2 le recevra déclaré SPAM.
Le sujet du courrier sera complété du tag \*\*\*SPAM\*\*\*.
Fin

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Vérifiez sur debian10-vm2 que clientmail-vm2 le reçoit bien notifié du tag \*\*\*SPAM\*\*\* :

[![Capture - Thunderbird : Tag ***SPAM*** inclus dans le sujet](/wp-content/uploads/2021/04/thunderbird-detection-spam-430x149.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/thunderbird-detection-spam.jpg)

Thunderbird : Tag \*\*\*SPAM\*\*\* inclus dans le sujet

Une ligne amavis ... Passed SPAM ... doit être présente dans le fichier /var/log/mail.log.

\[srvdmz@srvdmz:~$\] sudo cat /var/log/mail.log | grep amavis

[![Capture - Amavisd-New : SPAM mis en quarantaine](/wp-content/uploads/2021/04/quarantaine-spam-580x44.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/quarantaine-spam.jpg)

Amavisd-New : SPAM mis en quarantaine

#### _11.2 - Détection automatique d'un VIRUS_

Par défaut, les VIRUS détectés sont mis en quarantaine dans un sous-dossier de /var/lib/amavis/virusmails/ et non transmis aux destinataires.

L'administrateur est en revanche informé de la présence et mise en quarantaine des virus.

Vous allez demander que les destinataires soient aussi informés de cette mise en quarantaine.

Editez pour cela le fichier 50-user :

\[srvdmz@srvdmz:~$\] sudo nano /etc/amavis/conf.d/50-user

et ajoutez ces 2 lignes juste après $max\_servers :

\# Destinataires notifiés de l'existence d'un virus
$warnvirusrecip = 1;

Redémarrez ensuite le service amavis :

\[srvdmz@srvdmz:~$\] sudo systemctl restart amavis

Envoyez à présent, depuis srvlan, un courrier de srvlan vers clientmail-vm1 contenant ceci :

X5O!P%@AP\[4\\PZX54(P^)7CC)7}$EICAR-STANDARD-\\
ANTIVIRUS-TEST-FILE!$H+H\*

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Vérifiez que postmaster et clientmail-vm1 reçoivent bien une information de détection de virus.

[![Capture - Thunderbird : Information de virus détecté](/wp-content/uploads/2021/04/thunderbird-detection-virus-430x203.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/thunderbird-detection-virus.jpg)

Thunderbird : Information de virus détecté

Une ligne montrant amavis ... Blocked INFECTED ... et une autre quarantine: h/virus ... doivent être présentes dans /var/log/mail.log. Le h/virus signifie une mise en quarantaine dans /var/lib/amavis/virusmails/h/.

[![Capture - Amavisd-New : VIRUS mis en quarantaine](/wp-content/uploads/2021/04/quarantaine-virus-580x42.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2021/04/quarantaine-virus.jpg)

Amavisd-New : VIRUS mis en quarantaine

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Voilà !  
Le courrier indésirable est géré.  
La partie 3 vous attend pour  
traiter le WebMail avec Roundcube ...

[Partie 3](/postfix-amavis-roundcube-debian10/)
