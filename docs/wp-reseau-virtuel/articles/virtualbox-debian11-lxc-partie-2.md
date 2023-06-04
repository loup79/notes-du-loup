---
title: "OVS - LXC / Debian 11 : 2/2"
date: "2021-09-20"
categories: 
  - "openvswitch-lxc"
---

## Mémento 5.12 - Conteneurs LXC

Vous allez à présent créer un conteneur plus sécurisé dit non privilégié, celui-ci diminuant le risque de voir l'utilisateur root du conteneur obtenir les privilèges complets sur l'hôte LXC.

Le conteneur non privilégié se veut plus isolé de l'hôte LXC qu'un conteneur privilégié.

### 2 - LXC _(Création d'un conteneur non privilégié)_

#### _2.1 - Création de l'utilisateur propriétaire_

Les conteneurs non privilégiés contrairement aux conteneurs privilégiés sont créés et accessibles sans avoir besoin d'être root ou un utilisateur du groupe sudo.  
  
Ceci est plus sécurisé car l'on ne peut pas devenir root sur l'hôte même si l'on réussit à sortir du conteneur.

Générez donc un utilisateur normal de nom switchctn2 :

\[switch@ovs:~$\] sudo adduser switchctn2

Un MDP pour cet utilisateur vous sera demandé.

Son dossier de nom switchctn2 a été créé dans /home/.

Affichez ensuite les UID et GID attribués à celui-ci :

\[switch@ovs:~$\] sudo cat /etc/s\*id | grep switchctn2

Retour :

```
switchctn2:165536:65536       # Plage UID
switchctn2:165536:65536       # Plage GID
```

UID = Identifiant Utilisateur  
GID = Identifiant Groupe

Vérifiez que la valeur du paramètre suivant est à 1 :

\[switch@ovs:~$\] sudo sysctl kernel.unprivileged\_userns\_clone

Si OK, le système permettra aux utilisateurs normaux d'exécuter des conteneurs non privilégiés.

Autorisez switchctn2 à créer et connecter 1 interface réseau en créant le fichier lxc-usernet :

\[switch@ovs:~$\] sudo nano /etc/lxc/lxc-usernet

et en y entrant le contenu suivant :

switchctn2 veth br2 1

soit 1 interface veth sur le bridge br2 d'Open vSwitch.

#### _2.2 - Création d'un conteneur de nom ctn2_

Connectez-vous maintenant en tant que switchctn2 :

\[switch@ovs:~$\] su - switchctn2

Créez un fichier de configuration par défaut pour LXC :

\[switchctn2@ovs:~$\] mkdir -p .config/lxc
\[switchctn2@ovs:~$\] nano .config/lxc/default.conf

et entrez le contenu suivant :

lxc.idmap = u 0 165536 65536       \# plage UID de switchctn2
lxc.idmap = g 0 165536 65536       \# plage GID de switchctn2

lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up

lxc.apparmor.profile = generated
lxc.apparmor.allow\_nesting = 1

Seules les 2 premières lignes ont été ajoutées par rapport au fichier utilisé pour créer ctn1.

Quittez à présent la session utilisateur switchctn2 :

\[switchctn2@ovs:~$\] exit

Installez acl pour une gestion avancée des droits Linux :

\[switch@ovs:~$\] sudo apt install acl

et affectez ces permissions aux fichiers de switchctn2 :

sudo setfacl -m u:165536:x /home/switchctn2
sudo setfacl -m u:165536:x /home/switchctn2/.local
sudo setfacl -m u:165536:x /home/switchctn2/.local/share

Puis créez le conteneur ctn2 avec la même Cde que celle utilisée pour générer ctn1 :

\[switch@ovs:~$\] su - switchctn2

\[switchctn2@ovs:~$\] DOWNLOAD\_KEYSERVER=\\
"keyserver.ubuntu.com" \\
lxc-create -n ctn2 -t download -- -d debian -r bullseye -a amd64

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Le résultat de la Cde lxc-create doit donner ceci :

![Capture - LXC : Conteneur ctn2 créé sans échec](/wp-content/uploads/2021/12/lxc-creation-ctn2.jpg)

LXC : Conteneur ctn2 créé sans échec

La distribution téléchargée a été mise en cache dans :  
/home/switchctn2/.cache/lxc/download/

Vérifiez la création et le statut stoppé du conteneur :

\[switchctn2@ovs:~$\] lxc-ls -f

![Capture - LXC : Statut du conteneur ctn2](/wp-content/uploads/2021/12/lxc-statut-ctn2.jpg)

LXC : Statut du conteneur ctn2

et afficher sa configuration par défaut :

\[switchctn2@ovs:~$\] cat .local/share/lxc/ctn2/config

![Capture - LXC : Configuration de base du conteneur ctn2](/wp-content/uploads/2021/12/lxc-config-base-ctn2.jpg)

LXC : Configuration de base du conteneur ctn2

#### _2.3 - Configuration réseau du conteneur_

La partie réseau, comme pour ctn1, doit être traitée avant de démarrer ctn2.

La configuration de ctn2 doit être modifiée pour raccorder celui-ci sur le bridge br2 d'OVS.

Configuration à créer :

![Image - LXC : Configuration conteneur ctn2 vers bridge br2 d'OVS](/wp-content/uploads/2021/12/lxc-ctn2-veth-ovs.jpg)

LXC : Configuration conteneur ctn2 vers bridge br2 d'OVS

Pour cela, éditez la configuration du conteneur :

\[switchctn2@ovs:~$\] nano .local/share/lxc/ctn2/config

et modifiez les lignes suivantes comme suit :

#lxc.apparmor.profile = generated
lxc.apparmor.profile = unconfined

# Network configuration
lxc.net.0.type = veth
#lxc.net.0.link = lxcbr0
lxc.net.0.link = br2
lxc.net.0.flags = up
lxc.net.0.ipv4.address = 192.168.3.8/24
lxc.net.0.ipv4.gateway = 192.168.3.15       \# IP enp0s3 VM ovs

La configuration réseau de ctn2 est maintenant prête.

Quittez la session utilisateur switchctn2 :

\[switchctn2@ovs:~$\] exit

#### _2.4 - Démarrage du conteneur_

Au préalable, entrez la Cde suivante :

\[switch@ovs:~$\] sudo loginctl enable-linger switchctn2

La Cde enable-linger activera les processus persistants pour l'utilisateur switchctn2.

Cela maintiendra ctn2 démarré en cas de fermeture de session utilisateur switchctn2 et permettra le démarrage automatique du conteneur ctn2 au boot de la VM ovs.

Vérifiez la prise en compte de la Cde :

\[switch@ovs:~$\] sudo loginctl list-users

![Capture - LXC : Persistance switchctn2 activée](/wp-content/uploads/2021/12/lxc-loginctl-users-ctn2.jpg)

LXC : Persistance switchctn2 activée

Pour ensuite démarrer ctn2, lancez la Cde suivante :

\[switch@ovs:~$\] su - switchctn2
\[switchctn2@ovs:~$\] lxc-unpriv-start -n ctn2

ou celle-ci pour disposer d'un fichier log de démarrage :

\[switchctn2@ovs:~$\] lxc-unpriv-start -n ctn2 \\
--logfile=ctn2-lxc.log --logpriority=TRACE

Le caractère \\ indique d'écrire le tout sur une seule ligne.

Vérifiez enfin le statut du conteneur :

\[switchctn2@ovs:~$\] lxc-ls -f

![Capture - LXC : Conteneur non privilégié ctn2 démarré](/wp-content/uploads/2021/12/lxc-demarrage-ctn2.jpg)

LXC : Conteneur non privilégié ctn2 démarré

et la création d'une seconde interface veth :

\[switchctn2@ovs:~$\] ip address

![Capture - LXC : Interface réseau veth1001...@if2 créée sur la VM ovs](/wp-content/uploads/2021/12/lxc-ovs-carte-veth-bis.jpg)

LXC : Interface réseau veth1001...@if2 créée sur la VM ovs

**Remarque** : Depuis Debian 11.3, l'adresse IPv4 peut ne pas apparaître du fait de l'intervention du service systemd-networkd activé dans le conteneur.

Pour régler ce problème, procédez comme suit :

\[switchctn2@ovs:~$\] lxc-unpriv-attach -n ctn2

\[root@ctn2:/#\] systemctl stop systemd-networkd
\[root@ctn2:/#\] systemctl disable systemd-networkd
\[root@ctn2:/#\] exit

\[switchctn2@ovs:~$\] exit

\[switch@ovs:~$\] sudo reboot
\[switch@ovs:~$\] su switchctn2

\[switchctn2@ovs:~$\] lxc-unpriv-start -n ctn2
\[switchctn2@ovs:~$\] lxc-ls -f

L'adresse IPv4 192.168.3.8 doit cette fois apparaître.

#### _2.5 - Exécution de Cdes dans le conteneur_

Le conteneur a été créé par défaut sans MDP root.

La Cde lxc-unpriv-attach permet d’exécuter des Cdes dans un conteneur non privilégié.

Utilisez donc celle-ci pour vous connecter sur ctn2 :

\[switchctn2@ovs:~$\] lxc-unpriv-attach -n ctn2

Un prompt root@ctn2:/# doit s'afficher.

Listez ensuite la configuration réseau de ctn2 :

\[root@ctn2:/#\] ip address

![Capture - LXC : Le conteneur montre une interface réseau eth0@if12](/wp-content/uploads/2021/12/lxc-ctn2-carte-eth0.jpg)

LXC : Le conteneur montre une interface réseau eth0@if12

Il faut, afin que ctn2 accède à Internet, lui préciser l'IP du serveur DNS à utiliser.

Pour cela, éditez le fichier DNS resolv.conf de ctn2 :

\[root@ctn2:/#\] vim /etc/resolv.conf

et ajoutez la ligne suivante :

nameserver 192.168.x.z    \# IP de la Box Internet réseau virtuel

Instructions pour utiliser l'éditeur de textes vim :  
Touche i _(mode insertion)_ > Pour entrez nameserv... .  
Touche Escape > Pour quitter le mode insertion.  
Touches :wq suivies de Entrée > Pour sauvegarder.  

Pour tester l'accès à Internet, installez l'éditeur nano :

\[root@ctn2:/#\] apt install nano

Enfin, utilisez la Cde exit pour quitter le conteneur.

**Remarque** : Depuis Debian 11.3, le nameserver peut être par la suite écrasé du fait de l'intervention du service systemd-resolved activé dans le conteneur.

Pour régler ce problème, procédez comme suit :

\[switchctn2@ovs:~$\] lxc-unpriv-attach -n ctn2

\[root@ctn2:/#\] systemctl stop systemd-resolved
\[root@ctn2:/#\] systemctl disable systemd-resolved

\[root@ctn2:/#\] rm /etc/resolv.conf
\[root@ctn2:/#\] nano /etc/resolv.conf

Entrez ce contenu dans le nouveau fichier resolv.conf :

nameserver 192.168.x.z    \# IP de votre Box Internet

Puis, quittez ctn2, rebootez ovs et redémarrez ctn2 :

\[root@ctn2:/#\] exit

\[switchctn2@ovs:~$\] exit

\[switch@ovs:~$\] sudo reboot
\[switch@ovs:~$\] su switchctn2

\[switchctn2@ovs:~$\] lxc-unpriv-start -n ctn2
\[switchctn2@ovs:~$\] lxc-ls -f

Entrez ensuite dans ctn2 et vérifiez si le nameserver du fichier resolv.conf est correct.

#### _2.6 - Démarrage automatique du conteneur_

Stoppez le conteneur ctn2 si démarré :

\[switchctn2@ovs:~$\] lxc-stop -n ctn2

Créez à présent un service systemd utilisateur de nom lxc-autostart-ctn2.service :

\[switchctn2@ovs:~$\] mkdir -p .config/systemd/user
\[switchctn2@ovs:~$\] nano .config/systemd/user/lxc-autostart-ctn2.service

et entrez le contenu suivant :

\[Unit\]
Description="Démarrage auto de ctn2"

\[Service\]
Type=oneshot
ExecStart=/usr/bin/lxc-unpriv-start -n ctn2
ExecStop=/usr/bin/lxc-stop -n ctn2
RemainAfterExit=1

\[Install\]
WantedBy=default.target

Le service incluant la Cde lxc-unpriv-start aura besoin pour fonctionner de lire la variable d'environnement XDG\_RUNTIME\_DIR concernant l'utilisateur switchctn2.

Pour cela, éditez le fichier caché .bashrc de switchctn2 :

\[switchctn2@ovs:~$\] nano .bashrc

et entrez les 3 lignes suivantes à la fin du fichier :

\# Initialisation variable XDG\_RUNTIME\_DIR pour switchctn2
export XDG\_RUNTIME\_DIR="/run/user/$UID"
export DBUS\_SESSION\_BUS\_ADDRESS="unix:path=${XDG\_RUNTIME\_DIR}/bus" 

Quittez la session switchctn2 et reconnectez-vous :

\[switchctn2@ovs:~$\] exit
\[switch@ovs:~$\] su - switchctn2

Démarrez maintenant le service lxc-autostart-ctn2 :

\[switchctn2@ovs:~$\] cd .config/systemd/user
\[switchctn2@ovs:~$\] systemctl --user start lxc-autostart-ctn2

et si OK, validez son activation au boot de la VM ovs :

\[switchctn2@ovs:~$\] systemctl --user enable lxc-autostart-ctn2

Puis rebootez la VM et contrôlez le statut de ctn2 :

\[switch@ovs:~$\] su - switchctn2
\[switchctn2@ovs:~$\] lxc-ls -f

![Capture - LXC : Démarrage automatique du conteneur ctn2](/wp-content/uploads/2021/12/lxc-ctn2-demarrage-auto.jpg)

LXC : Démarrage automatique du conteneur ctn2

#### _2.7 - Tests divers sur le réseau virtuel_

Testez depuis l'intérieur de ctn2 les pings suivants :

\[root@ctn2:/#\] ping mappy.fr                                \# Internet
\[root@ctn2:/#\] ping 192.168.2.1                      \# VM srvsec
\[root@ctn2:/#\] ping 192.168.4.2                     \# VM srvdmz
\[root@ctn2:/#\] ping 192.168.3.4         \# VM debian11-vm2
\[root@ctn2:/#\] ping 192.168.3.6              \# Conteneur ctn1

Tous doivent recevoir une réponse positive.

Pour finir, testez un ping depuis la VM srvsec sur ctn2 :

\[root@srvsec:~#\] ping 192.168.3.8              \# Conteneur ctn2

Si retour OK, c'est terminé.

![Image - Rédacteur satisfait](/wp-content/uploads/2021/08/redacteur_satisfait_ter.jpg "Image Pixabay - Mohamed Hassan")

  
Voilà pour les bases de LXC !  
Le mémento 6.11 vous attend pour  
découvrir l'accès à distance sur  
les VM et les conteneurs.

[Mémento 6.11](https://familleleloup.no-ip.org/acces-locaux-distants-debian11/)
