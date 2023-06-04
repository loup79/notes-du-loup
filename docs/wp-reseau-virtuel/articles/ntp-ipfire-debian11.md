---
title: "Serveur NTP / IPFire"
date: "2022-02-13"
categories: 
  - "serveur-srvsec"
---

## NTP _(Network Time Protocol)_

Utilisation du serveur NTP de IPFire par les VM VirtualBox et conteneurs LXC du réseau virtuel.

### 1 - Rôle que jouera le serveur NTP de IPFire

Il permettra aux VM et au conteneur LXC privilégié ctn1 de synchroniser leur horloge système avec la sienne, ceci à base de protocole NTP _(port UDP 123)_.

C'est, entre autres, l'utilisation de protocoles de sécurité tels TSIG et DNSSEC au sein des serveurs DNS et DHCP qui exige que le temps réseau soit synchronisé.

### 2 - Utilisation du temps sur les clients Debian

VirtualBox utilise par défaut l'horloge de son hôte comme base de synchronisation pour les VM.

Résultat :  
La synchro horaire normalement assurée par le service systemd-timesyncd des Debian est désactivée de force sur les VM graphiques, ceci dû à la gestion d'un conflit avec le service vboxadd-service fourni par VirtualBox.

Les configurations suivantes feront en sorte que les horloges système des VM et du conteneur ctn1 soient uniquement synchronisées par le serveur NTP de IPFire.

Seul le conteneur non privilégié ctn2 continuera d'être synchronisé par son hôte soit la VM ovs.

### 3 - Configuration du serveur NTP sur IPFire

Connectez-vous sur l'interface Web d'IPFire depuis la VM srvlan _(ref : [§ 1.3 du Mémento 2.1](/serveur-ipfire-srvsec-creation/#13_-_Connexion_sur_linterface_WEB_dIPFire))_ .

\- Ensuite, menu IPFire  
\> Services > Heure du serveur > Paramètres communs  
\> Serveur NTP Primaire > Entrez 0.fr.pool.ntp.org  
\> Serveur NTP Secondaire > Entrez 1.fr.pool.ntp.org  
\> Cochez : Fournir l'heure au réseau local  
\> Bouton Sauvegarder

[![Capture - IPFire : Page Web de configuration du serveur NTP](/wp-content/uploads/2022/02/ipfire-serveur-ntp-580x278.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/02/ipfire-serveur-ntp.webp)

IPFire : Page Web de configuration du serveur NTP

Les modifications sauvegardées ci-dessus ont été automatiquement inscrites dans le fichier de configuration suivant : /var/ipfire/time/settings.

Etrangement, le fichier de configuration ntp.conf doit être modifié manuellement.

Pour cela, installez au préalable l'éditeur de texte nano :  
\- Toujours, menu IPFire  
\> IPFire > Pakfire > Modules disponibles  
\> Sélectionnez nano > Signe + pour l'installer

Une fois fini, nano apparaît dans Modules installés.

Editez ensuite, depuis la VM srvsec le fichier ntp.conf :

\[root@srvsec ~\]# nano /etc/ntp.conf

et modifiez son contenu comme suit :

disable monitor
restrict default nomodify noquery
restrict 127.0.0.1
server 0.fr.pool.ntp.org server 1.fr.pool.ntp.org server  127.127.1.0 
fudge   127.127.1.0 stratum 10
driftfile /etc/ntp/drift

Puis redémarrez le service ntp :

\[root@srvsec ~\]# /etc/init.d/ntp restart

Attendez quelques minutes et lancez la Cde ntpq -pn :

\[root@srvsec ~\]# ntpq -pn

[![Capture - IPFire : Retour de la Cde ntpq -pn](/wp-content/uploads/2022/02/ipfire-cde-ntpq-580x96.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/02/ipfire-cde-ntpq.webp)

IPFire : Retour de la Cde ntpq -pn

Détail des paramètres du tableau retourné :  
\- remote = IP/nom du serveur NTP distant interrogé.  
\- refid = source de synchronisation du remote _(ici .GPS.)_.  
\- st = strate du remote, valeur de 1 à 16 _(très bon = 1)_.  
\- poll = remote interrogé toutes les 64 secondes.  
\- reach = niveau d'accessibilité du remote, max = 377. 
\- delay = durée de la requête NTP exprimée en ms.  
\- offset = écart en ms de l'heure avec celle du remote.

\- `*` désigne le serveur choisi comme étant le plus fiable.  
\- `+` indique un serveur crédible, mais non sélectionné.  
  
L'horloge système .LOCL. de strate 10, paramètre fudge du fichier ntp.conf, ne sera jamais utilisée sauf incident.

Pour accéder à celle-ci, ntp utilise l'IP 127.127.1.0. Cette horloge peut servir de secours en cas de problème NTP.

Lancez maintenant la Cde ntpq -c as :

\[root@srvsec ~\]# ntpq -c as

[![Capture - IPFire : Résultat Cde ntpq -c as](/wp-content/uploads/2022/02/ipfire-cde-ntpq-bis-580x128.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/02/ipfire-cde-ntpq-bis.webp)

IPFire : Retour de la Cde ntpq -c as

\- sys.peer = source d'horloge soit le remote 176.137.3... .

#### _3.1 - Règle de pare-feu pour la zone Orange_

La politique de base implantée dans le pare-feu d'IPFire fait que tous les paquets NTP transmis depuis la zone Orange sont bloqués d'office.

Autorisez donc ce trafic NTP comme suit :

\- Menu IPFire  
\> Pare-feu > Règles de pare-feu > Nouvelle règle

\- Source  
\> Cochez Réseaux standards > Sélectionnez ORANGE

\- Destination  
\> Cochez Firewall > Sélectionnez ORANGE

\- Protocole  
\> Sélectionnez \- Préétabli - 
\> Cochez Services > Sélectionnez NTP  
\> Cochez ACCEPTER

\- Paramètres additionnels  
\> Remarque  
\> Entrez Trafic NTP autorisé entre IPFire et Orange

\> Ajouter > Appliquer les changements

[![Capture - IPFire : NTP autorisé entre IPFire et zone Orange](/wp-content/uploads/2022/02/ipfire-ntp-ok-orange-580x114.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/02/ipfire-ntp-ok-orange.webp)

IPFire : NTP autorisé entre IPFire et zone Orange

### 4 - Configuration du client NTP sur les VM

Au préalable, mettez à jour les VM :

$ sudo apt update  
$ sudo apt upgrade

et vérifiez le statut de la synchronisation horaire :

$ timedatectl status

Retour = inactive sur les VM graphiques sinon active :

```
Local time: mer. 2022-02-02 12:37:39 CET
Universal time: mer. 2022-02-02 11:37:39 UTC
RTC time: mer. 2022-02-02 11:37:39
Time zone: Europe/Paris (CET, +0100)
System clock synchronized: no ou yes (VM ovs)
NTP service: inactive ou active (VM ovs)
RTC in local TZ: no
```

Vérifiez le statut du service systemd-timesyncd :

$ systemctl status systemd-timesyncd

Retour = idem selon que la VM soit graphique ou non.

La différence de statut vient de l'utilisation ou non du service vboxadd-service de VirtualBox _(Voir plus haut)_.

Au final, le plus simple pour homogénéiser le tout est de désactiver la synchronisation horaire assurée par VirtualBox puis de se passer du service systemd-timesyncd sur toutes les VM au profit du service ntp.

#### _4.1 - Arrêt de la gestion horaire par VirtualBox_

Stoppez les VM et désactivez pour chacune la synchronisation horaire gérée par VirtualBox.

Exemple depuis un hôte Windows, sous PowerShell :

PS C:\\Users\\...> cd "C:\\Program Files\\Oracle\\VirtualBox\\"

PS C:\\Users\\...> .\\VBoxManage setextradata "nom VM" \\
"VBoxInternal/Devices/VMMDev/0/Config\\
/GetHostTimeDisabled" 1

Le caractère \\ indique d'écrire la Cde sur une seule ligne.

Exemple depuis un hôte Linux, sous Terminal :

$ vboxmanage setextradata "nom VM" \\
"VBoxInternal/Devices/VMMDev/0/Config\\
/GetHostTimeDisabled" 1

Redémarrez ensuite l'ensemble des VM.

#### _4.2 - Arrêt du service systemd-timesyncd_

Désactivez le service systemd-timesyncd comme suit :

$ sudo systemctl disable --now systemd-timesyncd
$ sudo systemctl stop systemd-timesyncd

Ne pas supprimer le paquet à cause des dépendances.

#### _4.3 - Activation et test du service ntp_

Commencez par installer le paquet ntp :

$ sudo apt install ntp

Retour :

```
Lecture des listes de paquets... Fait
Construction de l'arbre des dépendances... Fait
Lecture des informations d'état... Fait      
Les paquets supplémentaires ... seront installés : 
  libevent-core-2.1-7 libevent-pthreads-2.1-7 ...
Paquets suggérés :
  ntp-doc
Les paquets suivants seront ENLEVÉS :
  systemd-timesyncd
Les NOUVEAUX paquets suivants seront installés :
  libevent-core-2.1-7 libevent-pthreads-2.1-7 ...
0 mis à jour, 5 nouv..., 1 à enlever et 0 non ...
Il est nécessaire de prendre 1 221 ko dans les ...
Après cette opération, 2 842 ko d'espace disque ...
Souhaitez-vous continuer ? [O/n] o
```

Le paquet systemd-timesyncd sera retiré isolément.

Modifiez ensuite le fichier de configuration ntp.conf :

$ sudo nano /etc/ntp.conf

en commentant les 4 lignes suivantes :

pool \*.debian.pool.ntp.org iburst

et en les remplaçant par ceci pour les VM côté LAN :

server 192.168.2.1  \# IP Green de IPFire

ou par ceci pour la VM côté DMZ :

server 192.168.4.1  \# IP Orange de IPFire

Enfin, redémarrez le service ntp :

$ sudo systemctl restart ntp

et contrôlez la prise en compte du serveur NTP d'IPFire :

$ ntpq -pn

[![Capture - VM cliente : IPFire utilisé comme serveur NTP  (ntpq -pn)](/wp-content/uploads/2022/02/vm-ntp-actif-ipfire-580x70.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/02/vm-ntp-actif-ipfire.webp)

VM cliente : IPFire utilisé comme serveur NTP (ntpq -pn)

$ ntpq -c as

[![Capture - VM cliente : IPFire utilisé comme serveur NTP (ntpq -c as)](/wp-content/uploads/2022/02/vm-ntp-actif-ipfire-bis-580x94.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/02/vm-ntp-actif-ipfire-bis.webp)

VM cliente : IPFire utilisé comme serveur NTP (ntpq -c as)

La valeur du offset, soit le décalage avec l'horloge d'IPFire, tendra progressivement vers 0 au bout de quelques échanges de trames NTP avec le serveur.

#### _4.4 - Contrôle de la bonne gestion du temps_

Installez l'outil complémentaire ntpstat :

$ sudo apt install ntpstat

et contrôlez le statut de la synchronisation horaire :

$ ntpstat

![Capture - VM cliente : IPFire utilisé comme serveur NTP (ntpstat)](/wp-content/uploads/2022/02/vm-ntpstat.webp)

VM cliente : IPFire utilisé comme serveur NTP (ntpstat)

Effectuez le même test mais avec la Cde timedatectl :

$ timedatectl status

![Capture - VM cliente : Retour de la Cde timedatectl status](/wp-content/uploads/2022/02/vm-timedatectl.webp)

VM cliente : Retour de la Cde timedatectl status

Résultat : Le démon ntpd du client est bien synchronisé.

### 5 - Configuration du client NTP sur les conteneurs

#### _5.1 - Conteneur LXC privilégié ctn1_

Au préalable, mettez à jour le conteneur :

\[root@ctn1:/#\] apt update
\[root@ctn1:/#\] apt upgrade

Puis, comme ci-dessus, désactivez systemd-timesyncd :

\[root@ctn1:/#\] systemctl disable --now systemd-timesyncd
\[root@ctn1:/#\] systemctl stop systemd-timesyncd

et installez cette fois un concurrent de ntp soit chrony :

\[root@ctn1:/#\] apt install chrony

Afin que ce service fonctionne correctement, vous devez modifier les capacités _(fonctionnalités_ du noyau Linux) attribuées à l'utilisateur root de ctn1.

Créez pour cela, sur ovs, un fichier 01-sys-time.conf :

\[switch@ovs:~$\] cd /usr/share/lxc/config/common.conf.d
\[switch@ovs:~$\] sudo nano 01-sys-time.conf

et entrez le contenu suivant :

\# Modification des capacités du conteneur ctn1
# pour pouvoir utiliser NTP avec l'outil chrony

lxc.cap.drop =

lxc.cap.drop = mac\_admin mac\_override sys\_module sys\_rawio

Pour les curieux, les valeurs par défaut se trouvent dans /usr/share/lxc/config/common.conf.

Maintenant, stoppez et redémarrez le conteneur ctn1 :

\[switch@ovs:~$\] sudo lxc-stop -n ctn1
\[switch@ovs:~$\] sudo lxc-start -n ctn1

Puis, sur celui-ci, vérifiez le statut du service chrony :

\[root@ctn1:/#\] systemctl status chrony

Si OK, éditez son fichier de configuration chrony.conf :

\[root@ctn1:/#\] nano /etc/chrony/chrony.conf

Commentez la ligne suivante :

pool 2.debian.pool.ntp.org iburst

et ajoutez celle-ci juste en dessous :

server 192.168.2.1 iburst

Ajustez le fuseau horaire qui sera utilisée par ctn1 :

\[root@ctn1:/#\] timedatectl set-timezone "Europe/Paris"

Redémarrez enfin le service chrony :

\[root@ctn1:/#\] systemctl restart chrony
\[root@ctn1:/#\] systemctl status chrony

et testez la Cde chronyc tracking :

[![Capture - NTP : Retour de la Cde chronyc tracking](/wp-content/uploads/2022/02/cde-chronyc-tracking-580x294.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/02/cde-chronyc-tracking.webp)

NTP : Retour de la Cde chronyc tracking

ainsi que la Cde chronyc sources :

[![Capture - NTP : Retour de la Cde chronyc sources](/wp-content/uploads/2022/02/cde-chronyc-sources-580x67.webp "Cliquez pour agrandir l'image")](/wp-content/uploads/2022/02/cde-chronyc-sources.webp)

NTP : Retour de la Cde chronyc sources

et pour finir la Cde timedatectl show :

![Capture - NTP : Retour de la Cde timedatectl show](/wp-content/uploads/2022/02/cde-timedatectl-show.webp)

NTP : Retour de la Cde timedatectl show

#### _5.2 - Conteneur LXC non privilégié ctn2_

_Nota : les horloges système des conteneurs sont par défaut synchronisées sur celle de l'hôte._

Un conteneur non privilégié se voulant plus isolé de l'hôte LXC donc plus sécurisé de base qu'un conteneur privilégié, vous ne modifierez pas cette fois les capacités attribuées à l'utilisateur root de ctn2.

L'hôte LXC soit ovs est de toute façon actuellement synchronisé par le serveur NTP d'IPFire.

Il est par contre nécessaire d'ajuster le fuseau horaire :

$ timedatectl set-timezone "Europe/Paris"

\---------- Fin ----------
