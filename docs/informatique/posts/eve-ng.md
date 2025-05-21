---
title: EVE-NG - Homelab Réseau
summary: Ajout d'une VM dans un labo EVE-NG.
authors: 
  - G.Leloup
date: 2024-03-24
categories: 
  - EVE-NG
---

## Ajout d'une VM Debian

EVE-NG est installé sur une base Proxmox 8.x.

### Préparation

Se rendre dans le dossier prévu pour les images QEMU :  

```bash
cd /opt/unetlab/addons/qemu/
```

et créer celui qui recevra l'image de la VM :

```bash
sudo mkdir linux-nom
```

Puis transférer dans ce dossier une image ISO de Debian, ceci à l'aide de FileZilla en protocole SFTP.

Pour l'authentification SFTP, choisir l'utilisateur _root_ d'EVE-NG + le MDP de _root_.

Une fois l'image ISO transférée, renommer celle-ci :

<!-- more -->

```bash
sudo mv nom-iso cdrom.iso
```

et la convertir en image QEMU _(Ext .qcow2)_ comme suit :

```bash
sudo /opt/qemu/bin/qemu-img create -f qcow2 virtioa.qcow2 12G
```

12G = Taille souhaitée du disque dur.

Vérifier le processeur supportant EVE-NG :

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

### Accès Internet (pnet1)

L'exploitation du noeud Réseau _pnet0 (Cloud0)_ pour accéder à Internet ne fonctionne pas sous Proxmox et VirtualBox.

Utiliser _pnet1_ à la place de _pnet0_ nécessite de configurer EVE-NG.

Activer le routage interne en éditant le fichier _sysctl.conf_ :

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

Menu Add an object -> _Node_  
\- Sélectionner le template _Linux_  
\- Sélectionner l'image nommée _linux-nom_  
\- Configurer les paramètres _CPU_ _RAM_ et _Ethernets_  
\- Cliquer sur le bouton _Save_

Relier la VM au noeud Réseau _pnet1 (Cloud1)_.

Il suffit ensuite d'un clic droit sur l'icône du noeud + _Start_ pour démarrer la VM et procéder à son installation et sa configuration.

### Configuration de la VM

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

### Validation de la VM

Relever l'ID du labo courant (Menu de gauche -> Lab details) et l'ID du noeud créé  (Menu de l'icône -> Edit).

Se rendre ensuite dans le dossier _ID-du-labo/ID-du-noeud/_, exemple :

```bash
cd /opt/unetlab/tmp/0/02d314a2-8ab2-43...../2/
```

_0_ est l'identifiant du créateur du labo courant.

Puis, lancer la Cde suivante :

```bash
sudo /opt/qemu/bin/qemu-img commit virtioa.qcow2
```

L'image QEMU du dossier _linux-nom_ est alors remplacée par celle de la VM installée/configurée.

Le fichier ISO initial peut maintenant être supprimé :

```bash
rm -f /opt/unetlab/addons/qemu/linux-nom/cdrom.iso
```

### Noms d'hôtes et utilisateurs

Si l'on crée un nouveau noeud à partir de l'image validée ci-dessus, celui-ci héritera des paramètres comme le nom d'hôte et le nom de l'utilisateur principal.

Sur une Debian, procéder comme suit pour modifier ces 2 paramètres :

\- Hôte

```bash
sudo hostnamectl set-hostname nouveau-nom
sudo nano /etc/hosts
```

Remplacer dans _hosts_ l'ancienne valeur par _nouveau-nom_.

Pour finir, redémarrer la VM et relancer la Cde _hostnamectl_ pour vérifier la prise en compte de la modification.

\- Utilisateur principal

Se déconnecter du bureau LXQt puis ouvrir une console _root_ avec la combinaison de touches _CTRL+ALT+F2_ et entrer les 2 Cdes suivantes :

```bash
usermod -d /home/le-nouveau-nom -m -l le-nouveau-nom ancien-nom
passwd le-nouveau-nom
```

Puis, stopper le noeud depuis l'interface Web du labo, démarrer à nouveau celui-ci et se connecter sur la VM à l'aide du nouvel utilisateur _le-nouveau-nom_ et de son MDP.

### Configuration réseau

Le paramétrage réseau de la VM sous LXQt s'effectue à l'aide de l'outil graphique Connman _(Service connman)_.

Les fichiers de configuration des interfaces réseau se trouvent dans le dossier _/var/lib/connman/_.

Retirer éventuellement une mauvaise configuration réseau dans le fichier _/etc/network/interfaces ( service networking )_.

Vérifier également la configuration du fichier _/etc/hosts_.

### Taille du disque virtuel

Il est possible de redimensionner la taille du disque virtuel de la VM à l'aide de la Cde suivante, ceci VM stoppée :

```bash
cd /opt/unetlab/tmp/0/02d314a2-8ab2-43...../2/
sudo qemu-img resize virtioa.qcow2 +4G
```

+4G signifie que l'on demande une extension de 4 Go.

Redémarrer la VM, installer l'outil graphique _gparted_ sur celle-ci et redimensionner les partitions du disque virtuel afin d'exploiter pleinement la nouvelle taille.

**Fin.**
