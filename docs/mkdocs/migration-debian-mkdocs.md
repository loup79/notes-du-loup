---
title: MkDocs - Migration Debian
summary: Migration d'une Debian x vesr une Debian y.
author: G.Leloup
date: 2026-09-23
---

## Exemple Debian 12 vers Debian 13

VMs sur Mini-PC S51 et NAS Qnap _(code-server + mkdocs + thème material for mkdocs)_.

### Préparation

MkDocs se situe dans un environnement virtuel Python.  
La Cde mkdocs serve se lance manuellement.  
Aucun service systemd de Debian ne dépend directement de MkDocs.

Visual Studio Code Server est installé depuis un paquet Debian téléchargé de nom code-server_4.13x.0_amd64.deb.

#### Sauvegardes des VM

Faire une sauvegarde des 2 VM avant de commencer la migration.

#### Mise à jour de Debian 12

Mettre à jour le système Linux courant :

```bash
sudo apt update
sudo apt upgrade
sudo cat /etc/*version*
sudo reboot
```

#### Infos Python et MkDocs

Lister le contenu du dossier MkDocs courant :

```bash
cd /chemin/dossier-mkdocs
source env/bin/activate
ls
```

Retour normal :

```markdown
docs env mkdocs.yml requirements.txt
```

Vérifier ensuite les versions de Python et MkDocs courantes :

```bash
python --version
mkdocs --version
pip freeze > requirements.txt
```

La Cde pip freeze sert à lister tous les packages installés dans l’environnement Python actif avec leurs versions exactes, afin de générer un fichier requirements.txt pour recréer ou partager un même environnement sur d’autres machines.

Retours possibles :

```markdown
Python 3.11.2
mkdocs, version 1.6.1 ...
```

Autres informations issues du fichier requirements.txt :

```bash
grep -E '^(mkdocs|mkdocs-material|mkdocs-material-extensions|pymdown-extensions)' requirements.txt
```

Retour :

```markdown
mkdocs==1.6.1
mkdocs-get-deps==0.2.2
mkdocs-glightbox==0.3.7
mkdocs-material==9.7.6
mkdocs-material-extensions==1.3.1
pymdown-extensions==10.21.3
```

PyMdown Extensions est une collection d’extensions pour Python Markdown.

#### Test de la Cde build --strict

Tester un build sur le Mini-PC S51 :

```bash
sudo /chemin/dossier-mkdocs/env/bin/mkdocs build -c --strict
```

et sur le NAS Qnap :

```bash
sudo /chemin/dossier-mkdocs/env/bin/mkdocs build -d /var/www/html/mkdocs/ --strict
```

Seulement si OK, sortir de l'environnement virtuel Python :

```bash
deactivate
```

### Migration Debian

#### Vérifications avant migration

Vérifier le contenu des dossiers apt et sources.list.d :

```bash
sudo ls -l /etc/apt/
sudo ls -l /etc/apt/sources.list.d/
```

Le dossier sources.list.d est normalement vide.

Vérifier l'existence de paquets ne faisant pas partie du dépôt Debian :

```bash
apt list '?narrow(?installed, ?not(?origin(Debian)))'
```

Retour normal :

```markdown
code-server/now 4.135.0 amd64  [installé, local]
....
Autres paquets éventuels
....
```

Vérifier la liste des paquets dont la mise à jour est éventuellement bloquée (holding) :

```bash
sudo apt-mark showhold
```

La cde ne devrait normalement rien retourner.

Vérifier les paquets éventuellement cassés :

```bash
sudo dpkg --audit
```

La cde ne devrait normalement rien retourner.

Faire ceci si une réparation est nécessaire :

```bash
sudo apt --fix-broken install
sudo dpkg --configure -a
sudo apt full-upgrade
```

Vérifier enfin que l'espace disque disponible est suffisant :

```bash
sudo df -h
sudo df -h /boot
```

#### Migration vers Debian 13

Remplacer les valeurs bookworm par trixie dans le fichier /etc/apt/sources.list comme suit :

```markdown
deb http://deb.debian.org/debian/ trixie main non-free-firmware non-free

deb http://security.debian.org/debian-security trixie-security main non-free-firmware

deb http://deb.debian.org/debian/ trixie-updates main non-free-firmware
```

Vérifier de ne pas utiliser le dépôt bookworm-backports, à défaut commenter la ligne concernée. Le dépôt pourra être réactivé et mis à jour après la migration.

Effectuer ensuite la migration :

```bash
sudo apt update
sudo apt upgrade --without-new-pkgs
sudo apt full-upgrade
```

et redémarrer le nouveau Debian :

```bash
sudo reboot
```

#### Vérifications après migration

Vérifier la nouvelle version et si il y a des paquets Debian cassés :

```bash
sudo cat /etc/debian_version
sudo dpkg --audit
```

Un problème avec udev et udisk2 est signalé sur le Qnap en raison de sa version de noyau trop ancienne pour Debian 13.

#### Problème avec udev et udisk2

Dans le conteneur du Qnap, tester l'existence du groupe 997 :

```bash
getent group 997
grep '^[^:]*:[^:]*:997:' /etc/group
```

Si pas de retour ci-dessus, créer le groupe 997 comme suit :

```bash
sudo groupadd --system --gid 997 clock
getent group clock
getent group 997
```

Les retours des Cdes getend doivent montrer ceci :

```markdown
clock:x:997:
clock:x:997:
```

Mettre ensuite à jour les utilisateurs et groupes Debian :

```bash
cd /etc
sudo systemd-sysusers
echo $?
```

Retour normal :

```markdown
0
```

Réparer l'installation du paquet udev :

```bash
loup@debian-lxd-qnap:/etc$ sudo dpkg --configure udev
```

Retour normal :

```markdown
Paramétrage de udev (257.13-1~deb13u1) ...
Removing obsolete conffile /etc/init.d/udev ...
systemd-udevd.service is a disabled or a static unit not running, not starting it.
```

Réparer l'installation du paquet udisk2 :

```bash
loup@debian-lxd-qnap:/etc$ sudo dpkg --configure udisks2
```

Retour normal :

```markdown
Paramétrage de udisks2 (2.10.1-12.1+deb13u2) ...
Installation de la nouvelle version du fichier de configuration /etc/udisks2/mount_options.conf.example ...
Installation de la nouvelle version du fichier de configuration /etc/udisks2/udisks2.conf ...
```

Vérifier la prise en compte des 2 réparations :

```bash
sudo dpkg --audit
```

Le retour doit maintenant être vide.

### Python après migration

Vérifier la version Python du nouveau système Debian :

```bash
python3 --version
```

Retour possible 3.13.5.

Vérifier l'installation des paquets suivants, à défaut installer les manquants :

```bash
sudo apt list python3-pip python3-dev python3-venv python3-full
```

Ne pas réutiliser l'ancien env Python 3.11 de Debian 12, le renommer comme suit :

```bash
cd /chemin/dossier-mkdocs
mv env env-python311
```

Puis créer un nouvel environnement virtuel Python comme suit :

```bash
cd /home/user
mkdir -p /home/user/chemin/dossier-mkdocs/env
cd /home/user/chemin/dossier-mkdocs
python3 -m venv env
source env/bin/activate
cd env
python -m pip install --upgrade pip
cd ..
pip install -r requirements.txt
ls -lh requirements.txt
```

Le contenu du fichier requirements.txt permet la réinstallation de mkDocs.

### MkDocs après migration

Vérifier la version :

```bash
mkdocs --version
```

Tester un build sur le Mini-PC S51 :

```bash
sudo /chemin/dossier-mkdocs/env/bin/mkdocs build -c --strict
```

et sur le NAS Qnap :

```bash
sudo /chemin/dossier-mkdocs/env/bin/mkdocs build -d /var/www/html/mkdocs/ --strict
```

Si le build passe, tester ensuite la Cde mkdocs serve.

!!! note "Nota"
    Attention au chargement du module Apache PHP sur le Mini-PC S51 qui passe de 8.2 à 8.4. Il est nécessaire de corriger afin que le site Web fonctionne de nouveau comme avant. Apache n'est pas installé sur le Qnap.

Pour finir, faire un test d'upgrade de mkdocs et son thème depuis le nouvel environnement virtuel Python :

```bash
pip show mkdocs-material
pip install --upgrade --force-reinstall mkdocs-material
pip show mkdocs-material
```

Si tout est OK le dossier env-python311 créé ci-dessus peut être supprimé.

### Git

Faire un git status sur le Qnap afin de vérifier que tout est OK après la migration.

**Fin.**
