---
title: "E-mail / Debian 11 : 2/3"
date: "2022-10-26"
categories: 
  - "serveur-de-courrier"
---

## Mémento 11.21 - Postscreen / Rspamd

Le filtrage du courrier indésirable fera cette fois appel aux outils antispam Postscreen et Rspamd plus récents que SpamAssassin ainsi qu'à l'antivirus ClamAV.

### 10 - Filtrage du courrier indésirable

Le filtrage exigera de la ressource mémoire, ClamAV en sera le principal consommateur.

Pour info, ressource mémoire actuelle :

\[srvdmz@srvdmz:~$\] free --mega -t

|  | total | utilisé | libre |
| --- | --- | --- | --- |
| Mem: | 1023 | 560 | 164 |
| Partition d'échange: | 1022 | 106 | 916 |
| Total | 2046 | 667 | 1080 |

Constat : ≈ 1Go de libre incluant la partition d'échange.

Augmentez, si possible, la RAM actuelle de 1Go à 4 Go.

Les 3 Go d'écart seront utilisés par Clamav, à défaut celui-ci ne fonctionnera pas correctement et il sera préférable de ne pas traiter la partie antivirus.

Vous revérifierez cette ressource à la fin du mémento.

#### _10.1 - Installation de l'antispam Postscreen_

Postscreen, placé en amont de smtpd, rejettera entre autres les connexions issues des spambots avant que celles-ci ne viennent polluer le serveur Postfix.

Les spambots sont des robots à l'origine de la majorité des SPAM émis sur Internet.

Pour activer Postscreen, éditez le fichier master.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/master.cf

Commentez la ligne suivante _(# devant la ligne)_ :

smtp  inet          n     - y     - - smtpd

et décommentez celles-ci _(suppression du #)_ :

smtp  inet          n     - y      - 1      postscreen
smtpd pass       - - y      - - smtpd
dnsblog  unix    - - y      - 0      dnsblog
tlsproxy unix     - - y      - 0      tlsproxy

#### _10.2 - Postscreen : Configuration_

Pour le configurer, éditez le fichier main.cf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et entrez le contenu suivant en fin de fichier :

\# Filtrage avec Postscreen
postscreen\_access\_list = permit\_mynetworks, cidr:/etc/postfix/postscreen\_access.cidr
postscreen\_blacklist\_action = drop

postscreen\_greet\_wait = 3s
postscreen\_greet\_banner = Bienvenue et merci de patienter ...
postscreen\_greet\_action = enforce

postscreen\_dnsbl\_threshold = 3
postscreen\_dnsbl\_action = enforce
postscreen\_dnsbl\_sites =
        zen.spamhaus.org\*3
        b.barracudacentral.org=127.0.0.\[2..11\]\*2
        bl.spameatingmonkey.net\*2
        bl.spamcop.net
        #dnsbl.sorbs.net
        swl.spamhaus.org\*-4,
        list.dnswl.org=127.\[0..255\].\[0..255\].0\*-2,
        list.dnswl.org=127.\[0..255\].\[0..255\].1\*-4,
        list.dnswl.org=127.\[0..255\].\[0..255\].\[2..3\]\*-6

postscreen\_dnsbl\_whitelist\_threshold = -2

postscreen\_non\_smtp\_command\_enable = yes
postscreen\_non\_smtp\_command\_action = enforce

postscreen\_pipelining\_enable = yes
postscreen\_pipelining\_action = enforce

postscreen\_bare\_newline\_enable = yes
postscreen\_bare\_newline\_action = enforce

Le test access\_list permettra de filtrer les clients SMTP selon la valeur des IP contenues dans le paramètre mynetworks et le fichier postscreen\_access.cidr.

Ce dernier doit être créé et rempli manuellement :

\[srvdmz@srvdmz:~$\] cd /etc/postfix/
\[srvdmz@srvdmz:~$\] sudo touch postscreen\_access.cidr

Ci-dessous, un exemple de contenu _(vide par défaut)_ :

\# Adresses IP autorisées
192.168.2.2	permit

# Adresses IP rejetées
164.52.24.168	reject
71.6.158.166	reject
71.6.146.186	reject
109.206.241.42	reject
...

Le test greet permettra de rejeter les clients SMTP qui parleront avant que le serveur ne les y autorise comme l'exige le protocole SMTP.

Les tests dnsbl _(Black List DNS)_ et dnswl _(White List DNS)_ permettront de filtrer à l'aide de listes fournies sur Internet les clients SMTP considérés comme fiables ou susceptibles de transmettre des SPAMS.

_Nota : Postscreen maintient une liste blanche dynamique dans le fichier postscreen\_cache.db situé dans /var/lib/postfix._

Le test non\_smtp permettra de rejeter les clients qui useront de Cdes non SMPT telles connect, get et post.

Le test pipelining permettra de rejeter les clients ne respectant pas le protocole d'envoi de Cdes SMTP par lot.

Le test bare\_newline permettra de rejeter les clients ne respectant pas la syntaxe SMTP de fin de ligne.

Pour terminer, redémarrez Postfix :

\[srvdmz@srvdmz:~$\] sudo systemctl restart postfix

Vous devriez rapidement constater l'efficacité de Postscreen en éditant les logs de Postfix :

\[srvdmz@srvdmz:~$\] sudo cat /var/log/mail.log | grep postfix

Retour :

```
Oct 15 16:12:29 srvdmz postfix/postscreen[6598]: CONNECT from [37.139.128.50]:54706 to [192.168.4.2]:25

Oct 15 16:12:29 srvdmz postfix/dnsblog[6601]: addr 37.139.128.50 listed by domain zen.spamhaus.org as 127.0.0.4

Oct 15 16:12:29 srvdmz postfix/postscreen[6598]: PREGREET 11 after 0.04 from [37.139.128.50]:54706: EHLO User\r\n

Oct 15 16:12:30 srvdmz postfix/postscreen[6598]: DNSBL rank 3 for [37.139.128.50]:54706

Oct 15 16:12:30 srvdmz postfix/postscreen[6598]: DISCONNECT [37.139.128.50]:54706
```

Si IP blacklistée dans le fichier postscreen\_access.cidr :

```
Oct 15 16:14:20 srvdmz postfix/postscreen[7216]: CONNECT from [37.139.128.50]:63279 to [192.168.4.2]:25

Oct 15 16:14:20 srvdmz postfix/postscreen[7216]: BLACKLISTED [37.139.128.50]:63279

Oct 15 16:14:20 srvdmz postfix/postscreen[7216]: DISCONNECT [37.139.128.50]:63279
```

#### _10.3 - Installation de l'antispam Rspamd_

Rspamd, plus récent et moderne que SpamAssassin, viendra compléter la lutte antispam.

Installez le paquet rspamd :

\[srvdmz@srvdmz:~$\] sudo apt install rspamd

Son installation inclut celle d'un serveur de Bdd Redis.

Tous les fichiers de configuration ont été créés dans le dossier /etc/rspamd/.

Vérifiez enfin le statut des services rspamd et redis :

\[srvdmz@srvdmz:~$\] sudo systemctl status rspamd
\[srvdmz@srvdmz:~$\] sudo systemctl status redis

Retours :

[![Capture - Rspamd et Redis : Statuts](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-redis-debian11-430x293.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-redis-debian11.webp)

Rspamd et Redis : Statuts

L'intégration du logiciel Rspamd dans Postfix se fera via le protocole Milter.

Pour activer celle-ci, éditez le fichier main.cf de Postfix :

\[srvdmz@srvdmz:~$\] sudo nano /etc/postfix/main.cf

et modifiez la section \# Application Milter (DKIM) comme montré ci-dessous :

\# Applications Milter (DKIM, RSPAMD, ...)
smtpd\_milters = inet:localhost:8891, inet:127.0.0.1:11332
non\_smtpd\_milters = $smtpd\_milters
milter\_protocol = 6
milter\_default\_action = accept
milter\_mail\_macros = i {mail\_addr} {client\_addr} {client\_name} {auth\_authen}

Rechargez enfin la nouvelle configuration de Postfix :

\[srvdmz@srvdmz:~$\] sudo systemctl reload postfix

L'outil linux opendkim étant actif, désactivez le module DKIM fourni par Rspamd comme suit :

\[srvdmz@srvdmz:~$\] cd /etc/rspamd/local.d
\[srvdmz@srvdmz:~$\] sudo nano dkim\_signing.conf

Entrez ce contenu dans le fichier dkim\_signing.conf :

\# Désactivation du module DKIM de Rspamd
enabled = false;

Ceci évitera l'entrée d'erreurs dans les logs de Rspamd.

Redémarrez Rspamd :

\[srvdmz@srvdmz:~$\] sudo systemctl restart rspamd

#### _10.4 - Rspamd : _Auto-apprentissage bayésien__

Activez l'auto-apprentissage bayésien en créant le fichier classifier-bayes.conf :

\[srvdmz@srvdmz:~$\] cd /etc/rspamd/override.d
\[srvdmz@srvdmz:~$\] sudo nano classifier-bayes.conf

et en y insérant les lignes suivantes :

autolearn = true;

# Envoi des données statistiques vers le serveur Redis
users\_enabled = true;
backend = "redis";

Le filtrage bayésien est une technique de probabilité permettant de traiter le courrier indésirable.

Créez ensuite un fichier redis.conf :

\[srvdmz@srvdmz:~$\] sudo nano redis.conf

et indiquez la localisation du serveur Redis comme suit :

servers = "127.0.0.1";

Demandez à Rspamd de taguer le courrier indésirable en créant un fichier milter\_headers.conf :

\[srvdmz@srvdmz:~$\] sudo nano milter\_headers.conf

et en y insérant l'instruction suivante :

extended\_spam\_headers = true;

Pour classer automatiquement ce courrier tagué X-Spam: Yes, vous ferez appel au plugin sieve de Dovecot.

Editez, pour cela, le fichier 90-sieve.conf de Dovecot :

\[srvdmz@srvdmz:~$\] cd /etc/dovecot/conf.d
\[srvdmz@srvdmz:~$\] sudo nano 90-sieve.conf

et ajoutez ceci sous la ligne #sieve\_after = :

sieve\_after = /etc/dovecot/sieve-after

Créez ensuite le dossier sieve-after :

\[srvdmz@srvdmz:~$\] sudo mkdir /etc/dovecot/sieve-after

et ajoutez dans celui-ci un fichier 10-spam.sieve :

\[srvdmz@srvdmz:~$\] cd /etc/dovecot/sieve-after
\[srvdmz@srvdmz:~$\] sudo nano 10-spam.sieve

contenant la règle Sieve suivante :

require \["fileinto","mailbox"\];

if header :contains "X-Spam" "Yes" {
fileinto :create "Junk";
stop;
}

L'utilisation de la règle implique de compiler celle-ci :

\[srvdmz@srvdmz:~$\] sudo sievec 10-spam.sieve

Un fichier binaire 10-spam.svbin a été généré.

Redémarrez maintenant Dovecot et Rspamd :

\[srvdmz@srvdmz:~$\] sudo systemctl restart dovecot
\[srvdmz@srvdmz:~$\] sudo systemctl restart rspamd

La syntaxe des fichiers de configuration de Rspamd peut être contrôlée comme suit :

\[srvdmz@srvdmz:~$\] sudo rspamadm configtest

Retour :

```
Syntax OK
```

#### _10.5 - Rspamd : Apprentissage selon actions ..._

Si un utilisateur déplace un courrier dans le dossier des spams, Rspamd apprendra que c’est un spam et si un utilisateur déplace un courrier du dossier des spams vers un dossier autre que la corbeille, Rspamd apprendra que c'est un ham soit le contraire d'un spam.

Cet apprentissage impose l'usage du plugin imap\_sieve, éditez pour cela le fichier 20-imap.conf :

\[srvdmz@srvdmz:~$\] cd /etc/dovecot/conf.d
\[srvdmz@srvdmz:~$\] sudo nano 20-imap.conf

et ajoutez ceci sous #mail\_plugins = $mail\_plugins :

protocol imap {          \# Ligne existante

mail\_plugins = $mail\_plugins imap\_sieve

}                                    \# Ligne existante

Editez ensuite le fichier 90-sieve.conf :

\[srvdmz@srvdmz:~$\] sudo nano 90-sieve.conf

et entrez le contenu suivant à la fin du fichier :

plugin {         \# Ligne existante

# Apprentissage selon actions utilisateurs

    sieve\_plugins = sieve\_imapsieve sieve\_extprograms

    # Si déplacé dans le dossier spam > script learn-spam.sieve
    imapsieve\_mailbox1\_name = Junk
    imapsieve\_mailbox1\_causes = COPY
    imapsieve\_mailbox1\_before = file:/etc/dovecot/sieve/learn-spam.sieve

    # Si retiré du dossier spam > script learn-ham.sieve
    imapsieve\_mailbox2\_name = \*
    imapsieve\_mailbox2\_from = Junk
    imapsieve\_mailbox2\_causes = COPY
    imapsieve\_mailbox2\_before = file:/etc/dovecot/sieve/learn-ham.sieve

    # Dossier contenant les fichiers de scripts
    sieve\_pipe\_bin\_dir = /etc/dovecot/sieve

    # Autoriser l'envoi des courriers vers un programme externe
    sieve\_global\_extensions = +vnd.dovecot.pipe +vnd.dovecot.environment

}                      \# Ligne existante

Créez le dossier sieve dédié aux scripts Sieve/Rspamd :

\[srvdmz@srvdmz:~$\] sudo mkdir /etc/dovecot/sieve

Générez le 1er script Sieve learn-spam.sieve :

\[srvdmz@srvdmz:~$\] cd /etc/dovecot/sieve/
\[srvdmz@srvdmz:~$\] sudo nano learn-spam.sieve

et entrez le contenu suivant proposé par Dovecot :

require \["vnd.dovecot.pipe", "copy", "imapsieve", "environment", "variables"\];

if environment :matches "imap.user" "\*" {
  set "username" "${1}";
}

pipe :copy "rspamd-learn-spam.sh" \[ "${username}" \];

Générez enfin le 2ème script Sieve learn-ham.sieve :

\[srvdmz@srvdmz:~$\] sudo nano learn-ham.sieve

et entrez le contenu suivant proposé par Dovecot :

require \["vnd.dovecot.pipe", "copy", "imapsieve", "environment", "variables"\];

if environment :matches "imap.mailbox" "\*" {
  set "mailbox" "${1}";
}

if string "${mailbox}" "Trash" {
  stop;
}

if environment :matches "imap.user" "\*" {
  set "username" "${1}";
}

pipe :copy "rspamd-learn-ham.sh" \[ "${username}" \];

Redémarrez Dovecot avant de continuer :

\[srvdmz@srvdmz:~$\] sudo systemctl restart dovecot

puis compilez les scripts Sieve et modifiez leurs droits :

\[srvdmz@srvdmz:~$\] sudo sievec learn-spam.sieve
\[srvdmz@srvdmz:~$\] sudo sievec learn-ham.sieve

\[srvdmz@srvdmz:~$\] sudo chmod u=rw,go= /etc/dovecot/sieve/learn-{spam,ham}.{sieve,svbin}

\[srvdmz@srvdmz:~$\] sudo chown vmail:vmail /etc/dovecot/sieve/learn-{spam,ham}.{sieve,svbin}

Générez le 1er script Rspamd rspamd-learn-spam.sh :

\[srvdmz@srvdmz:~$\] sudo nano rspamd-learn-spam.sh

et entrez le contenu suivant :

#!/bin/sh
exec /usr/bin/rspamc learn\_spam

Générez le 2ème script Rspamd rspamd-learn-ham.sh :

\[srvdmz@srvdmz:~$\] sudo nano rspamd-learn-ham.sh

et entrez le contenu suivant :

#!/bin/sh
exec /usr/bin/rspamc learn\_ham

Modifiez les droits des 2 fichiers et relancez Dovecot :

\[srvdmz@srvdmz:~$\] sudo chmod u=rwx,go= /etc/dovecot/sieve/rspamd-learn-{spam,ham}.sh

\[srvdmz@srvdmz:~$\] sudo chown vmail:vmail /etc/dovecot/sieve/rspamd-learn-{spam,ham}.sh

\[srvdmz@srvdmz:~$\] sudo systemctl restart dovecot

Pour tester la configuration, respectez ces 6 étapes :

**1)** Affichez le dossier Indésirables sur les Thunderbird.  
\> Clic droit sur les comptes ...@loupvirtuel.fr  
\> Paramètres > Paramètres des indésirables  
\> Cochez Déplacer les nouv... indésirables vers  
\> Sélectionnez Dossier << Indésirables >> sur  
\> ...@loupvirtuel.fr  
  
Faites de même pour le compte x.y@zzz.freeddns.org.

**2)** Activez le mail\_debug de /etc/dovecot/conf.d/10-logging.conf à yes et relancez Dovecot.

**3)** Envoyez un mail de srvdmz vers srvlan et déplacez le mail reçu dans le dossier Indésirables.

**4)** Déplacez ensuite ce mail du dossier Indésirables vers le dossier Courrier entrant de srvlan.

**5)** Observez enfin le contenu des logs de /var/log/mail.log et /var/log/rspamd/rspamd.log.

[![Capture - Logs : Lignes présentes de Dovecot et Rspamd](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/actions-spam-ham-debian11-580x237.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/actions-spam-ham-debian11.webp)

Logs : Lignes présentes de Dovecot et Rspamd

**6)** Terminez en remettant la valeur du paramètre mail\_debug à no et relancez Dovecot.

#### _10.6 - Rspamd : Interface Web_

Rspamd est fourni avec une interface Web permettant de contrôler les courriers marqués comme spam, d'avoir des statistiques, etc...

Il est nécessaire, pour accéder à celle-ci, d'éditer l'hôte virtuel loupvirtuel.conf :

\[srvdmz@srvdmz:~$\] cd /etc/apache2/sites-available
\[srvdmz@srvdmz:~$\] sudo nano loupvirtuel.conf

et d'ajouter ceci dans la section <VirtualHost \*:443> :

ProxyPass "/rspamd" "http://localhost:11334"
ProxyPassReverse "/rspamd" "http://localhost:11334"

Redémarrez le serveur Web Apache :

\[srvdmz@srvdmz:~$\] sudo systemctl restart apache2

Créez enfin le MDP d'accès exigé par l'interface Web :

\[srvdmz@srvdmz:~$\] sudo rspamadm pw

Exemple de retour :

```
Enter passphrase: Entrez votre MDP
$2$qr5xkz9rrwrcrd3kw7ba6a9p5wezo146......
```

Créez maintenant un fichier worker-controller.inc :

\[srvdmz@srvdmz:~$\] cd /etc/rspamd/local.d
\[srvdmz@srvdmz:~$\] sudo nano worker-controller.inc

Entrez le hachage du MDP comme suit :

password = "$2$qr5xkz9rrwrcrd3kw7ba6a9p5wezo146......"

et relancez rspamd :

\[srvdmz@srvdmz:~$\] sudo systemctl restart rspamd

Pour finir, testez l'URL https://loupvirtuel.fr/rspamd/

[![Capture - Rspamd : Interface Web](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-site-web-debian11-430x243.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-site-web-debian11.webp)

Rspamd : Interface Web

#### _10.7 - Installation du scanner de virus ClamAV_

Par curiosité, contrôlez de nouveau les ressources disponibles qui ont du peu évoluer :

\[srvdmz@srvdmz:~$\] free --mega -t
\[srvdmz@srvdmz:~$\] df /dev/sda1 -H

Installez maintenant les paquets ClamAV suivants :

\[srvdmz@srvdmz:~$\] sudo apt install clamav clamav-daemon
\[srvdmz@srvdmz:~$\] sudo apt install clamav-unofficial-sigs

Un groupe/utilisateur clamav et 2 services ont été créés.

Le paquet clamav-unofficial-sigs installe une base virale fournie par la société SaneSecurity, base qui vient compléter celles de ClamAV.

Attendez 5 à 10 minutes et vérifiez le statut du service de MAJ des Bdd ClamAV :

\[srvdmz@srvdmz:~$\] sudo systemctl restart clamav-freshclam
\[srvdmz@srvdmz:~$\] sudo systemctl status clamav-freshclam

[![Capture - ClamAV : Statut du service clamav-freshclam](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/freshclam-statut-debian11-580x207.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/freshclam-statut-debian11.webp)

ClamAV : Statut du service clamav-freshclam

Les Bdd daily, main et bytecode doivent être up to date.

Si Bdd up to date, démarrez le service clamav-daemon :

\[srvdmz@srvdmz:~$\] sudo systemctl start clamav-daemon
\[srvdmz@srvdmz:~$\] sudo systemctl status clamav-daemon

[![Capture - ClamAV : Statut du service clamav-daemon](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/clamav-demon-status-debian11-580x239.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/clamav-demon-status-debian11.webp)

ClamAV : Statut du service clamav-daemon

Au prochain boot système, le service clamav-daemon démarrera automatiquement.

Le scan d'un dossier peut déjà s'effectuer comme suit :

\[srvdmz@srvdmz:~$\] sudo clamscan /home/srvdmz

[![Capture - ClamAV : Scan du dossier /home/srvdmz](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/clamav-scan-dossier-debian11-356x430.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/clamav-scan-dossier-debian11.webp)

ClamAV : Scan du dossier /home/srvdmz

Les logs sont consultables dans /var/log/clamav/.

Le bon traitement de la base virale clamav-unofficial-sigs peut être vérifié comme suit :

\[srvdmz@srvdmz:~$\] cd /home/srvdmz
\[srvdmz@srvdmz:~$\] sudo clamscan --debug 2>&1 /dev/null | grep "sanesecurity"

Retour :

```
LibClamAV debug: /var/lib/clamav/sanesecurity.ftm loaded
```

#### _10.8 - ClamAV_ : Configuration

Les fichiers sont situés dans le dossier /etc/clamav/.

Il est possible de réduire la fréquence de MAJ des Bdd en éditant le fichier freshclam.conf :

\[srvdmz@srvdmz:~$\] sudo nano /etc/clamav/freshclam.conf

et en modifiant la valeur 24 du paramètre Checks à 3 :

Checks 3

Les MAJ seront effectuées 3 fois par jour au lieu de 24.

_Nota : Comme pour root, créez depuis l'interface de PostfixAdmin un alias de l'utilisateur clamav vers postmaster afin que ce dernier puisse recevoir les notifications par e-mail de ClamAV._

#### _10.9 - ClamAV : Liaison avec Rspamd_

Afin que Rspamd utilise ClamAV, créez le fichier suivant :

\[srvdmz@srvdmz:~$\] sudo nano /etc/rspamd/local.d/antivirus.conf

et entrez le contenu ci-dessous :

clamav {
    scan\_mime\_parts = false;
    symbol = "CLAM\_VIRUS";
    type = "clamav";
    action = "reject";
    servers = "/var/run/clamav/clamd.ctl";
}

Pour finir rechargez la configuration de Rspamd :

\[srvdmz@srvdmz:~$\] sudo systemctl reload rspamd

### 11 - Tests

#### _11.1 - Test de détection d'un SPAM_

Au préalable, créez un fichier temporaire options.inc :

\[srvdmz@srvdmz:~$\] cd /etc/rspamd/local.d
\[srvdmz@srvdmz:~$\] sudo nano options.inc

et entrez le contenu suivant :

enable\_test\_patterns = true;

Ceci permettra d'effectuer le test attendu, voir les explications sur le site [rspamd.com](https://rspamd.com/doc/gtube_patterns.html).

Redémarrez ensuite Rspamd :

\[srvdmz@srvdmz:~$\] sudo systemctl restart rspamd

et envoyez, depuis le Thunderbird de srvlan, un courrier de postmaster vers clientmail-vm2 contenant ceci :

Début
YJS\*C4JDBQADN1.NSBN3\*2IDNEN\*GTUBE-STANDARD-\\
ANTI-UBE-TEST-EMAIL\*C.34X

Ce courrier doit normalement être traité comme spam.
La source de celui-ci sera complété du tag X-Spam: Yes.
Fin

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Si tout se passe bien, le courrier sera, sur clientmail-vm2, stocké automatiquement dans le dossier des Indésirables de Thunderbird.

Vérifiez sur debian11-vm2 que clientmail-vm2 le reçoit bien notifié du tag X-Spam: Yes :

[![Capture - Thunderbird : Tag X-Spam: Yes dans la source du mail](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-test-spam-debian11-430x203.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-test-spam-debian11.webp)

Thunderbird : Tag X-Spam: Yes dans la source du mail

Sur srvdmz, le courrier a été stocké dans :

\[srvdmz@srvdmz:~$\] sudo ls -la /var/mail/vmail/loupvirtuel.fr/clientmail-vm2/Maildir/.Junk/cur/

[![Capture - srvdmz : SPAM dans le dossier .Junk](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-stockage-spam-debian11-580x71.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-stockage-spam-debian11.webp)

srvdmz : SPAM dans le dossier .Junk

Le groupe date/heure correspond avec celui du mail reçu sur le compte virtuel clientmail-vm2.

Si OK, vous pouvez supprimer le fichier options.inc.

#### _11.2 - Test de détection d'un VIRUS_

Au préalable, éditez le fichier antivirus.conf :

\[srvdmz@srvdmz:~$\] cd /etc/rspamd/local.d
\[srvdmz@srvdmz:~$\] sudo nano antivirus.conf

et modifiez son contenu comme suit :

clamav {
    scan\_mime\_parts = false;
    symbol = "CLAM\_VIRUS";
    type = "clamav";
    action = "reject";
    servers = "/var/run/clamav/clamd.ctl";

patterns {
    JUST\_EICAR = '^Eicar-Test-Signature$';
                }
}

Ceci permettra d'effectuer le test attendu, voir les explications sur le site [rspamd.com](https://rspamd.com/doc/modules/antivirus.html).

Redémarrez ensuite Rspamd :

\[srvdmz@srvdmz:~$\] sudo systemctl restart rspamd

Envoyez à présent, depuis srvlan, un courrier de srvlan vers clientmail-vm1 contenant ceci :

X5O!P%@AP\[4\\PZX54(P^)7CC)7}$EICAR-STANDARD-\\
ANTIVIRUS-TEST-FILE!$H+H\*

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Ce contenu est téléchargeable depuis le site Eicar _(European Institute for Computer Antivirus Research)_ en utilisant l'URL [https://secure.eicar.org/eicar.com.txt](https://secure.eicar.org/eicar.com.txt).

Résultat, le virus sera détecté lors de la phase d'envoi du mail depuis srvlan et rejeté _(paramètre action = "reject"; du fichier antivirus.conf)_.

[![Capture - Thunderbird : Information de virus détecté](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-test-virus-debian11-430x214.webp "Cliquez pour agrandir l'image")](https://familleleloup.no-ip.org/wp-content/uploads/2023/03/rspamd-test-virus-debian11.webp)

Thunderbird : Information de virus détecté

#### _11.3 - Contrôle de la mémoire utilisée_

ClamAV est gourmand :

\[srvdmz@srvdmz:~$\] free --mega -t

|  | total | utilisé | libre |
| --- | --- | --- | --- |
| Mem: | 4122 | 2051 | 1386 |
| Partition d'échange: | 1022 | 220 | 802 |
| Total | 5145 | 2271 | 2188 |

Constat : ≈ 2Go de libre incluant la partition d'échange.

ClamAV exigera parfois plus que le 2051 Mo affiché ci-dessus d'où le besoin de 4 Go de RAM. A défaut n'utilisez pas le logiciel antivirus.

#### _11.4 - Contrôle des logs_

Consultez les fichiers /var/log/mail.log et /var/log/rspamd/rspamd.log, le premier vous permettant notamment d'observer l'efficacité de l'outil postscreen.

Vous allez vite comprendre que l'ouverture du port 25 sur votre box Internet sera rapidement exploitée par divers scanners de FAI et robots spammeurs.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà !  
Le courrier indésirable est géré.  
La partie 3 vous attend pour  
traiter le WebMail avec Roundcube ...

[Partie 3](https://familleleloup.no-ip.org/postfix-roundcube-debian11/)
