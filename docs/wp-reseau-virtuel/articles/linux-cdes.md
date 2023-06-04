---
title: "Linux - Lignes de commandes"
date: "2021-04-15"
categories: 
  - "cdes-linux"
---

## Aide mémoire des Cdes Linux

N'apparaissent que les Cdes utilisées pour les besoins de création et de test du réseau virtuel.

Vous pouvez utiliser apt-get à la place d'apt ou aptitude si ce gestionnaire est installé.

### 1 - Gestion des paquets

_1.1 - Mettre à jour un système Debian/Ubuntu_

\[root@xxxxx:~#\] apt update                   (MAJ liste des paquets) 
\[root@xxxxx:~#\] apt upgrade                         (MAJ des paquets) 
\[root@xxxxx:~#\] apt dist-upgrade (MAJ plus fine des paquets) 

_1.2 - Vider le cache contenant les paquets archivés_

\[root@xxxxx:~#\] apt clean                      (Gain d'espace disque)

Les paquets sont situés dans /var/cache/apt/archives/.

_1.3 - Supprimer les paquets installés devenus inutiles_

 \[root@xxxxx:~#\] apt autoremove --purge 

Sont également supprimés les dépendances devenues inutiles et les fichiers de configuration.

_1.4 - Vérifier si un paquet est installé ou non_

\[root@xxxxx:~#\] apt show nom-du-paquet

Retour > Une ligne APT-Manual-Installed: yes si installé.

Une autre Cde possible ci-dessous :

\[root@xxxxx:~#\] apt list nom-du-paquet

Exemple de retour si nom-du-paquet = nano :

```
En train de lister... Fait
nano/stable,now 5.4-2+deb11u1 amd64  [installé]
```

_1.5 - Installer un paquet_

\[root@xxxxx:~#\] apt install nano   (installe l'éditeur de textes) 

La liste des sources contenant les paquets téléchargeables se situe dans /etc/apt/sources.list.

_1.6 - Rechercher un paquet_

\[root@xxxxx:~#\] apt search nom-du-paquet

Trop de réponses, voir ce premier filtrage :

\[root@xxxxx:~#\] apt search --names-only nom-du-paquet

Encore trop de réponses, voir ce second filtrage :

\[root@xxxxx:~#\] apt search ^nom-du-paquet$

### 2 - Gestion des disques et mémoires

_2.1 - Afficher le % d'occupation du disque_

\[root@xxxxx:~#\] df 

_2.2 - Afficher la quantité de mémoire utilisée_

\[root@xxxxx:~#\] free ou free -m -t

_2.3 - Lister les disques et partitions actives_

\[root@xxxxx:~#\] fdisk -l

_2.4 - Lister les UUID disques et les PARTUUID partitions_

\[root@xxxxx:~#\] blkid

### 3 - Gestion des paramètres courants du PC

_3.1 - Afficher la version courante du noyau_

\[root@xxxxx:~#\] uname -r

_3.2 - Afficher le nom d'hôte_

\[root@xxxxx:~#\] hostname

_3.3 - Afficher la version de la distribution Linux installée_

\[root@xxxxx:~#\] cat /etc/issue

### 4 - Gestion des dossiers et fichiers

_4.1 - Création d'un fichier vide_

\[root@xxxxx:~#\] touch /chemin/nom-du-fichier

_4.2 - Renommer un fichier_

\[root@xxxxx:~#\] mv nom-du-fichier nouveau-nom-du-fichier

_4.3 - Vider le contenu d’un fichier log_

\[root@xxxxx:~#\] echo /dev/null > /chemin/nom-du-fichier-log

_4.4 - Supprimer un lien symbolique_

\[root@xxxxx:~#\] unlink nom-du-lien

_4.5 - Changer le propriétaire/groupe d'un fichier/dossier_

\[root@xxxxx:~#\] chown root:bind /chemin/nom.conf

Affecte le propriétaire root et le groupe bind à nom.conf.

_4.6 - Changer les permissions d'un fichier/dossier_

\[root@xxxxx:~#\] chmod 640 /chemin/nom.conf

6 = droits Propriétaire 4 = droits Groupe 0 = droits Autres  
7 = 4 + 2 + 1 = r + w + x = Lecture + Ecriture + Exécutable  
640 = r + w > Propriétaire, r > Groupe et rien > Autres

_4.7 - Créer un dossier_

\[root@xxxxx:~#\] mkdir /chemin/nom-du-dossier

### 5 - Gestion des applications

_5.1 - Lancement d'une app graphique en tant que root_

\[root@xxxxx:~#\] gksudo nom-application     (Cde obsolète) 

Utiliser maintenant la Cde pkexec du paquet PolicyKit-1.

### 6 - Gestion du réseau

_6.1 - Afficher la configuration des interfaces réseau_

\[root@xxxxx:~#\] ifconfig ou maintenant ip address

_6.2 - Vérifier l'activation ou non de l'adressage IPv6_

\[root@xxxxx:~#\] ip a | grep inet6

_6.3 - Désactiver/Activer une carte réseau, Ex : eth0_

\[root@xxxxx:~#\] ifdown eth0      (désactivation) 

\[root@xxxxx:~#\] ifup eth0               (activation) 

_6.4 - Tracer la route empruntée par un paquet IP_

\[root@xxxxx:~#\] traceroute adresse IP ou nom-de-domaine

### 7 - Gestion des services

_7.1 - Relancer un service_

Sous sysvinit :

\[root@xxxxx:~#\] restart network-manager              (réseau)  
\[root@xxxxx:~#\] service bind9 restart                         (DNS)  
\[root@xxxxx:~#\] service isc-dhcp-server restart      (DHCP) 
\[root@xxxxx:~#\] service apache2 restart                    (Web) 
\[root@xxxxx:~#\] service mysql restart                         (SQL)  

Sous systemd :

\[root@xxxxx:~#\] systemctl restart networking     (réseau_)_
\[root@xxxxx:~#\] systemctl restart bind9                   (DNS)

_7.2 - Afficher le niveau de démarrage d'un PC_

\[root@xxxxx:~#\] runlevel                 (Ex de résultat : N2) 

Regardez dans /etc/rc2.d l’ordre de démarrage des services de niveau 2 repérés Sxx....

En cas de problème identifié lors d'un boot, les numéros chronologiques Sxx... peuvent être manuellement modifiés pour changer l'ordre de démarrage.

_7.3 - Lister les services en cours d'exécution_

Sous sysvinit :

\[root@xxxxx:~#\] service --status-all 

Légendes des statuts retournés :  
+ = démarré, \- = stoppé, ? = inconnu.

Sous systemd :

\[root@xxxxx:~#\] systemctl list-units --type=service  

### 8 - Gestion des utilisateurs

_8.1 - Ajouter ou supprimer un utilisateur local_

\[root@xxxxx:~#\] adduser nom-utilisateur                    (ajout)  
\[root@xxxxx:~#\] deluser nom-utilisateur        (suppression)  

### 9 - Gestion des bdd MySQL ou MariaDB

_9.1 - Accéder au serveur SQL_

\[root@xxxxx:~#\] mysql -u nom-utilisateur -p -P 3306

3306 étant le numéro de port par défaut.

_9.2 - Lister les bdd existantes_

MariaDB \[(none)\] > show databases;

### 10 - Gestion d'un fichier Swap

_10.1 - Ajouter un fichier swap comme mémoire virtuelle_

a) Création d'un fichier de taille 3 Go :

\[root@xxxxx:~#\] dd if=/dev/zero of=/mnt/swapfile bs=1024 count=3145728

\[root@xxxxx:~#\] chmod 600 /mnt/swapfile

\[root@xxxxx:~#\] mkswap /mnt/swapfile

\[root@xxxxx:~#\] swapon /mnt/swapfile

Le fichier swapfile créé et activé dans /mnt viendra compléter la partition swap existante.

b) Déclaration dans le fichier des montages /etc/fstab :

\[root@xxxxx:~#\] nano /etc/fstab

Ajouter la ligne suivante :

/mnt/swapfile none            swap    sw              0       0

c) Redémarrer le système :

\[root@xxxxx:~#\] reboot

d) Vérifier la prise en compte du fichier swap :

root@xxxxx:~#\] swapon -s

Retour :

```
Nom de fichier    Type         Taille       Utilisé
/mnt/swapfile     file         3145724      2196800
/dev/vda5         partition    524284       0      
```

\---------- Fin ----------
