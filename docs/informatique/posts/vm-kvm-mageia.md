---
title: VM KMV - Mageia
summary: Installation du système Linux Mageia.
authors: 
  - G.Leloup
date: 2026-07-13
categories: 
  - VM KVM
---

Création d'une VM KVM pour Mageia 10 sur U59, seule distribution française encore vivante.

## Mémo Mageia

### Création de la VM

Utiliser la Cde suivante dans un terminal :

```bash
sudo virt-install \
  --name mageia10 \
  --ram 4096\
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/mageia10.qcow2,size=40,bus=virtio \
  --cdrom /mnt/ssd-interne/ISO/Mageia-Cauldron-netinstall-x86_64.iso \
  --os-variant mageia9 \
  --network bridge=br0,model=virtio \
  --graphics spice \
  --boot loader=/usr/share/OVMF/OVMF_CODE_4M.fd,loader.readonly=yes,loader.type=pflash,loader.secure=no \
  --noautoconsole
```

La VM créée ainsi démarre automatiquement, suivre ensuite la procédure d'installation du mode netinstall :

\- Partie mode Texte  
Méthode d'installation = FTP server -> OK  
Réseau = DHCP -> OK  
Nom d'hôte = mageia, rien pour le nom de domaine -> OK  
Proxy = rien -> OK  
Miroir = Mageia 10 -> OK  
Spécifier le miroir = geex.freeboxos.fr -> OK

Pas besoin de login et MDP pour le miroir.

<!-- more -->

\- Parttie mode Graphique  
Vers personnaliser le bureau pour pouvoir choisir Xfce.

Une fois l'installation en mode graphique terminée, stopper la VM et retirer le contenu du CDROM _(si nécessaire car à priori retiré automatiquement)_.

Redémarrer la VM et appliquer la configuration ci-dessous.

### Configuration de la VM

#### - _Groupe Wheel (sudo)_

Ajouter l'utilisateur principal au groupe Wheel :

```bash
su root
sudo usermod -aG wheel nom-du-user
reboot
```

L'utilisateur principal pourra ainsi utiliser la Cde sudo.

#### - _Serveur xrdp_

Installer le serveur xrdp :

```bash
sudo dnf install -y xrdp tigervnc-server
```

Vérifier la création d'un utilisateur xrdp avec la Cde id xrdp, à défaut créer celui-ci :

```bash
sudo useradd -r -s /sbin/nologin -M xrdp
```

Changer ensuite le propriétaire des fichiers suivants :

```bash
sudo chown xrdp:xrdp /etc/xrdp/cert.pem /etc/xrdp/key.pem /etc/xrdp/rsakeys.ini
```

Autoriser le démarrage automatique du service xrdp au boot du système :

```bash
sudo systemctl enable xrdp
sudo systemctl enable xrdp-sesman

sudo systemctl start xrdp
sudo systemctl start xrdp-sesman

sudo systemctl status xrdp
sudo systemctl status xrdp-sesman
```

Autoriser les connexions RDP sur l'utilisateur principal en éditant le fichier suivant :

```bash
sudo nano /etc/xrdp/sesman.ini
```

et en remplaçant la ligne Policy=Default par Policy=Allow.

Puis redémarrer le service xrdp-sesman :

```bash
sudo systemctl restart xrdp-sesman
```

Créer un script de démarrage pour la session graphique :

``` bash
sudo nano /etc/xrdp/startwm.sh
```

et entrer ce contenu :

```markdown
#!/bin/sh
xfce4-session
```

Rendre ensuite le script exécutable :

```bash
sudo chmod +x /etc/xrdp/startwm.sh
```

Une connexion sur le serveur xrdp devrait dorénavant fonctionner.

### Centre de contrôle Mageia

\- Le lanceur du Centre de Contrôle de Mageia n'ouvre rien si connecté en tant qu'utilisateur normal.

Pour y remédier :

```bash
cd /usr/share/applications
sudo nano mageia-drakconf.desktop
```

Remplacer Exec=/usr/bin/drakconf par Exec=pkexec /usr/bin/drakconf

Faire la même chose avec le lanceur /home/user/.local/share/applications/mageia-drakconf.desktop.

\- Le lanceur du Centre de Contrôle de Mageia n'ouvre rien si connecté **RDP** en tant qu'utilisateur normal.

Pour y remédier, créer le script suivant :

```bash
cd /home/loup
nano drakconf-pkexec.sh
```

et entrer ce contenu :

```markdown
#!/bin/bash
pkexec env DISPLAY=$DISPLAY XAUTHORITY=$XAUTHORITY /usr/bin/drakconf
```

Rendre le script exécutable :

```bash
chmod +x drakconf-pkexec.sh
```

Créer un lanceur propre à la session RDP :

```bash
nano .local/share/applications/drakconf-pkexec.desktop
```

Entrer le contenu suivant :

```markdown
[Desktop Entry]
Type=Application
Name=Centre de contrôle (RDP)
Comment[fr]=Centre de contrôle de Mageia
Exec=xfce4-terminal -e "/home/user/drakconf-pkexec.sh"
Icon=drakconf
Terminal=false
Categories=Settings;
```

L'icône créée peut-être déplacée dans la barre de menu du haut.

La configuration réseau de la VM peut se gérer depuis ce centre de contrôle.

### Serveur SSH

Installation :

```bash
sudo dnf install openssh-server
sudo systemctl start sshd
sudo systemctl status sshd
```

### Réglages du pare-feu

Un pare-feu de nom Shorewall est installé par défaut et actif, le vérifier ainsi :

```bash
sudo systemctl status shorewall
```

Ouvrir Menu -> Outils -> Centre de contrôle de Mageia  
-> Sécurité -> Configurer votre pare-feu personnel

Gérer le ping, le ssh et le port 3389.

Pour le port RDP 3389, cliquer sur Avancé et entrer 3389/tcp dans le champ Autres ports.

**Fin.**
