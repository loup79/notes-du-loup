---
title: EVE-NG - Homelab Réseau
summary: Ajout d'une VM debian dans un labo EVE-NG.
authors: 
  - G.Leloup
date: 2024-03-24
categories: 
  - EVE-NG
---

## Ajout d'une VM Debian

EVE-NG est une VM installée sur une base Proxmox 8.x.

### Préparation d'EVE-NG

Se rendre dans le dossier suivant :  

```bash
cd /opt/unetlab/addons/qemu/
```

et créer celui qui recevra l'image de la VM Debian :

```bash
sudo mkdir linux-nom
```

Transférer ensuite dans ce dossier l'image ISO de Debian à l'aide par exemple de FileZilla en protocole SFTP.

Pour l'authentification SFTP, choisir l'utilisateur _root_ d'EVE-NG + le MDP de _root_.

Une fois l'image ISO transférée, la renommer comme suit :

<!-- more -->

```bash
sudo mv nom-iso cdrom.iso
```

et la convertir en image QEMU _(Ext .qcow2)_ comme suit :

```bash
sudo /opt/qemu/bin/qemu-img create -f qcow2 virtioa.qcow2 12G
```

12G = Taille souhaitée du disque dur.

Trouver le type de processeur supportant EVE-NG :

```bash
lsmod | grep ^kvm_
```

Retour = _amd_ ou _intel_.

Puis ouvrir, selon le retour, le template associé _linux.yml_ :

```bash
sudo nano /opt/unetlab/html/templates/amd ou intel/linux.yml
```

et compléter la ligne _qemu_options_ avec le paramètre :

```markdown
-k fr
```

La VM disposera ainsi au démarrage d'un clavier français.

Une autre solution est d'ajouter directement le paramètre _-k fr_ dans le champ _QEMU custom options_ lors de la création de la VM Debian sous l'interface Web d'EVE-NG.

### Gestion de l'accès à Internet

L'exploitation du noeud Réseau _pnet0 (Cloud0)_ pour accéder à Internet ne fonctionne pas sous Proxmox et VirtualBox.

Utiliser _pnet1_ à la place de _pnet0_ est possible mais cela nécessite d'activer le routage interne en éditant le fichier de configuration _sysctl.conf_ :

```bash
sudo nano /etc/sysctl.conf
```

et en décommentant la ligne suivante :

```markdown
#net.ipv4.ip_forward = 1
```

Installer ensuite ces 2 paquets :

```bash
sudo apt install iptables-persistent netfilter-persistent
```

et créer la règle _iptables_ suivante :

```bash
sudo iptables -t nat -A POSTROUTING -o pnet0 -s 15.128.128.0/24 -j MASQUERADE
```

L'IP de _pnet1_ devra faire partie du réseau _15.128.128.0_.

Sauvegarder la nouvelle règle _iptables_ :

```bash
su root
sudo iptables-save > /etc/iptables/rules.v4
exit
```

et vérifier la bonne création de celle-ci :

```bash
sudo iptables -t nat -L
```

Finir en éditant le fichier réseau _interfaces_ :

```bash
sudo nano /etc/network/interfaces
```

et en modifiant la section _#Cloud device_ comme suit :

```markdown
# Cloud devices
iface eth1 inet manual
auto pnet1
iface pnet1 inet static
    bridge_ports eth1
    bridge_stp off
    address 15.128.128.1
    netmask 255.255.255.0
```

### Ajout de la VM dans un labo

Ajouter le VM Debian commme suit :

Menu Add an object -> _Node_  
\- Sélectionner le template _Linux_  
\- Sélectionner l'image nommée _linux-nom_  
\- Entrer un _nom-de-vm_ dans le champ Name/prefix  
\- Configurer les paramètres _CPU_ _RAM_ et _Ethernets_  
\- Ajouter _-k fr_ dans le champ _QEMU custom options_  
\- Laisser Console à _VNC_  
\- Cliquer sur le bouton _Save_

Relier la VM au noeud Réseau _pnet1 (Cloud1)_.

Il suffit ensuite d'un clic droit sur l'icône du noeud + _Start_ pour démarrer la VM et procéder à son installation.

### Installation de Debian sur la VM

Faire une installation de base avec une configuration manuelle du réseau et la mise en place des paquets  _serveur ssh_ et _utilitaires usuels_ seulement.

Après le premier reboot :

```bash
su root
apt update
apt upgrade
apt install sudo
sudo usermod -aG sudo user
/sbin/reboot
```

Vérifier ensuite que la carte réseau _ens3_ a bien été déclarée dans le fichier _/etc/network/interfaces_.

Vérifier en complément le retour de la Cde _ip a_.

Puis installer un environnement graphique xfce4 minimal :

```bash
sudo apt install xfce4
sudo reboot
```

Le système doit redémarrer sous xfce4.

Vérifier l'activation du serveur SSH :

```bash
sudo systemctl status sshd
```

Puis installer les applications suivantes :

```bash
sudo apt install xrdp
sudo adduser xrdp ssl-cert
sudo apt install network-manager
sudo apt install network-manager-gnome
sudo apt install xfce4-terminal mousepad
sudo apt install synaptic ristretto
sudo apt install xfce4-taskmanager
sudo apt install firefox-esr
sudo apt install firefox-esr-l10n-fr
sudo apt install gparted mtpaint
sudo poweroff
```

### Commit de l'installation Debian

Relever l'ID du labo courant (Menu de gauche -> Lab details) et l'ID du noeud créé  (Menu de l'icône -> Edit).

Se rendre ensuite dans le dossier _ID-du-labo/ID-du-noeud/_, exemple :

```bash
cd /opt/unetlab/tmp/0/02d314a2-8ab2-43...../2/
```

_0_ est l'identifiant du créateur du labo courant soit l'utilisateur _admin_ par défaut.

L'identifiant aura une valeur différente si la création du labo se fait depuis un utilisateur autre que _admin_.

Puis, lancer la Cde suivante :

```bash
sudo /opt/qemu/bin/qemu-img commit virtioa.qcow2
```

L'image QEMU du dossier _linux-nom_ est alors remplacée par celle de la VM Debian installée et configurée.

Le fichier ISO initial peut maintenant être supprimé :

```bash
rm -f /opt/unetlab/addons/qemu/linux-nom/cdrom.iso
```

On dispose maintenant d'une Debian de base prête à l'emploi _(installée et configurée)_ pour la création future d'un noeud _(VM)_ Debian sous EVE-NG.

### Configuration SSH et Telnet

Installer et activer le service _ssh_ :

```bash
sudo apt install openssh-server
```

Une VM d'IP _15.128.128.x_ et reliée au noeud Réseau _pnet1_ pourra ainsi être configurée directement depuis EVE-NG :

```bash
sudo ssh utilisateur-vm@ip-vm
```

Cela peut dépanner dans certains cas de figure.

Installer également le service _telnet_ dans le cas d'un serveur Debian :

```bash
sudo apt-get install xinetd telnetd
```

Créez pour cela un fichier de configuration _telnet_ :

```bash
sudo nano /etc/xinetd.d/telnet
```

et y entrer le contenu suivant :

```markdown
service telnet
{
disable = no
flags = REUSE
socket_type = stream
wait = no
user = root
server = /usr/sbin/telnetd
log_on_failure += USERID
}
```

Redémarrer le service _telnet_ :

```bash
sudo systemctl restart xinetd
```

Pour finir, activer une connexion série _ttyS0_ :

```bash
sudo systemctl enable serial-getty@ttyS0.service
```

EVE-NG pourra ainsi se connecter en Telnet sur le serveur Debian.

### Configuration réseau

Redémarrer la VM Debian depuis depuis l'interface Web d'EVE-NG, console VNC.

Ouvrir l'applet de _network-manager_ et créer ou modifier la carte réseau _ens3_

Puis :

```bash
sudo systemctl restart NetworkManager
```

Commenter les lignes du fichier /etc/network/interfaces.

Puis :

```bash
sudo systemctl restart networking
sudo reboot
```

Vérifier que tout est OK.

Si la VM comprend plusieurs interfaces réseau, affecter depuis network-manager une IP à chacune des interfaces.

\- Remarques sur LXQT à la place de XFCE -

Le paramétrage réseau d'une VM sous LXQT s'effectue à l'aide de l'outil graphique Connman _(Service connman)_.

Les fichiers de configuration des interfaces réseau se trouvent dans le dossier _/var/lib/connman/_.

\- Accès SSH

Penser à ajouter une route IP statique sur le PC local afin de pouvoir joindre le réseau pnet1.

Exemple pour Windows :

```markdown
route add -p 15.128.128.0 mask 255.255.255.0 192.168.x.y
```

\- Accès RDP

En console RDP, EVE-NG ajoute une carte réseau dédiée à la connexion RDP, ceci sans conséquence sur le bon fonctionnement de l'ensemble.

Penser à gérer les [Astuces XRDP](../../blog/posts/controle-distant-xrdp.md){ target="_blank" } permettant la modifications des interfaces réseau et l'arrêt de la VM Debian.

### Noms d'hôtes et utilisateurs

Si l'on crée un nouveau noeud à partir d'une image issue d'un commit comme ci-dessus, celui-ci héritera des paramètres comme le nom d'hôte et le nom de l'utilisateur principal.

Sur une Debian, procéder comme suit pour modifier ces 2 paramètres :

\- Hôte

```bash
sudo hostnamectl set-hostname new-host
sudo nano /etc/hosts
```

Remplacer dans _hosts_ l'ancienne valeur par _new-host_.

Pour finir, redémarrer la VM et relancer la Cde _hostnamectl_ pour vérifier la prise en compte de la modification.

\- Utilisateur principal

Se déconnecter du bureau graphique XFCE ou LXQT puis ouvrir une console _root_ avec la combinaison de touches _CTRL+ALT+F2_ et entrer les 2 Cdes suivantes :

```bash
usermod -d /home/new-user -m -l new-user old-user

passwd new-user

groupmod -n new-group old-group

chown -R new-user:new-group /home/new-user
```

Supprimer les dossiers _xfce4_ dans _/home/new-user/.config_ et _sessions_ dans _/home/new-user/.cache_.

```bash
rm -R /home/new-user/.config/xfce4
rm -R /home/new-user/.cache/sessions
reboot
```

Se connecter à présent sur la VM à l'aide du nouvel utilisateur _new-user_ et de son MDP.

Retirer le floppy disque et le cdrom du bureau xfce4 :  
\- Paramètres du bureau -> Icônes  
-> Icônes par défaut -> Périphériques amovibles  
-> décocher _Disques durs et lecteurs_

### Fond d'écran et CDROM

\- Fond d'écran  
Pour un même fond d'écran sur la fenêtre de login Lightdm et le bureau Xfce, procéder ainsi :

```bash
cd /chemin-fond-ecran-xfce4
sudo cp fond-ecran.jpg /usr/share/backgrounds/

cd /etc/lightdm
sudo nano lightdm-gtk-greeter.conf
```

Remplacer la ligne #background= par celle-ci :

```markdown
background=/usr/share/backgrounds/fond-ecran.jpg
```

```bash
sudo reboot
```

\- CDROM  
Démonter le contenu du cdrom0 utilisé pour la création de la VM.

Commenter la ligne faisant référence au montage du CDROM dans le fichier _/etc/fstab_.

Puis :

```bash
sudo systemctl daemon-reload
sudo systemctl restart remote-fs.target
```

### Taille du disque virtuel

Il est possible de redimensionner la taille du disque virtuel de la VM à l'aide de la Cde suivante, ceci VM stoppée :

```bash
cd /opt/unetlab/tmp/0/02d314a2-8ab2-43...../2/
sudo qemu-img resize virtioa.qcow2 +4G
```

+4G signifie que l'on demande une extension de 4 Go.

Redémarrer la VM et utiliser l'outil graphique _gparted_ pour redimensionner la partition du disque virtuel afin d'exploiter pleinement la nouvelle taille.

### Cas de l'utilisateur admin/ID 0

Si des VM ont été préalablement créées depuis l'utilisateur _admin_ mais que celui-ci n'existe plus, il est possible de supprimer ses fichiers temporaires devenus inutiles.

Vérifier par précaution s'il reste des fichiers récents ou s'il y a des processus en cours :

```bash
ls -lh /opt/unetlab/tmp/0/
ps aux | grep qemu
```

Si OK, procéder ainsi pour les supprimer :

```bash
sudo rm -rf /opt/unetlab/tmp/0/*
sudo find /opt/unetlab/tmp/0/ -mindepth 1 -delete
```

ATTENTION : Ne pas supprimer le dossier 0.

### Console HTML5

La connexion s'établie au travers d'un serveur Guacamole installé par défaut sur EVE-NG.

**Fin.**
