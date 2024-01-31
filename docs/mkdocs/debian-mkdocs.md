---
title: MkDocs - Site sur Debian
summary: Installation de MkDocs sur Debian.
author: G.Leloup
date: 2023-06-01
---

## MkDocs en local sur Debian

Tuto pour installer MkDocs sur une VM Debian.

### Environnement virtuel Python

Préalable : Python version 3 installé de base sur l'OS Debian.

Vérifier la version courante de Python :

```bash
python3 --version
```

et l'installation des modules Python suivants :

```bash
sudo apt list python3-pip python3-dev python3-venv python3-full
```

A défaut, installer les modules manquants à l'aide de la Cde _sudo apt install_.

Ensuite, créer un dossier qui recevra un ==environnement virtuel Python==, ceci pour ne pas interférer par la suite avec la version de Python installée sur l'OS :

```bash
cd /home/user
mkdir -p /home/user/Documents/notes-du-user/env
cd /home/user/Documents/notes-du-user
```

et installer l'environnement Python dans le dossier _env_ :

```bash
python3 -m venv env
```

Activer celui-ci afin de pouvoir l'exploiter :

```bash
cd env
source ./bin/activate
```

Le prompt du terminal Debian se change en ==_(env) user@deb..._==.

!!! note "Nota"
    Pour sortir de l'environnement Python, utiliser la Cde :  
    (env) deactivate

Rester dans l'environnement et mettre à jour le gestionnaire de paquets Python de nom _pip_ :

```bash
(env) user@deb...: python3 -m pip install --upgrade pip
```

### Installation de MkDocs

Installer MkDocs avec la Cde suivante :

```bash
(env) user@deb...: python3 -m pip install mkdocs
(env) user@deb...: mkdocs --version
```

Retour :

```markdown
mkdocs, version 1.5.3 from /home/user.../site-packages/mkdocs (Python 3.11)
```

Les fichiers de MkDocs sont installés dans :  
_/home/user/Documents/notes-du-user/env/lib/python3.xy/site-packages/mkdocs/_

Les thèmes de base sont installés dans :  
_/home/user/Documents/notes-du-user/env/lib/python3.xy/site-packages/mkdocs/themes/_

### Thème Material for MkDocs

Ajouter le thème comme ceci :

```bash
(env) user@deb...: python3 -m pip install mkdocs-material
```

Les dossiers et fichiers du thème sont installés dans :  
_/home/user/Documents/notes-du-user/env/lib/python3.xy/site-packages/material/_

!!! note "Nota"
    Lien GitHub du thème : [Material pour MkDocs](https://squidfunk.github.io/mkdocs-material/){ target="_blank" }

Pour vérifier la version du thème, utiliser cette Cde :

```bash
(env) user@deb...: pip show mkdocs-material
```

Retour :

```markdown
Name : mkdocs-material
Version : 9.5.5
...
```

Pour mettre à jour le thème, utiliser cette Cde :

```bash
(env) user@deb...: pip install --upgrade --force-reinstall mkdocs-material
```

### Projet/Dossier documentation

Créer le Projet/Dossier _(Site de développement)_ de la documentation à venir :

```bash
(env) user@deb...: cd /home/user/Documents/
(env) user@deb...: mkdocs new notes-du-user
(env) user@deb...: cd notes-du-user
(env) user@deb...: ls
```

Retour de la Cde _ls_ :

```markdown
docs env mkdocs.yml
```

Créer un fichier de nom _requirements.txt_ :

```bash
(env) user@deb...: cd /home/user/Documents/notes-du-user
(env) user@deb...: pip freeze > requirements.txt
```

Ce fichier permettrait à un développeur voulant travailller sur ce projet de connaitre les prérequis de l'environnement Python à utiliser, non nécessaire dans le cadre d'un site MkDocs local.

Activer le thème _Material for MkDocs_ en modifiant le fichier _mkdocs.yml_ comme suit :

```bash
(env) user@deb...: cd /home/user/Documents/notes-du-user
(env) user@deb...: nano mkdocs.yml
```

entrer le contenu suivant :

```markdown
site_name: Documents du user
dev_addr: '192.168.x.y:8000'

theme:
  name: material
  language: fr
  features:
    - navigation.footer

copyright: Copyright &copy; 2024 - Cartier Jacques
```

192.168.x.y = IP de la VM Debian.

### Serveur de développement

Pour lancer le serveur associé au site de développement, procéder ainsi :

```bash
(env) user@deb...: cd /home/user/Documents/notes-du-user
(env) user@deb...: mkdocs serve --dev-addr=192.168.x.y:8000
```

L'IP 192.168.1.x est celle de la VM Debian 11.

Retour normal :

```markdown
INFO     -  Building documentation...
INFO     -  Cleaning site directory
INFO     -  Documentation built in 1.20 seconds
INFO     -  [12:10:49] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO     -  [12:10:49] Serving on http://192.168.1.x:8000/
```

Touches _CTRL+C_ pour fermer le serveur.

### Serveur de production

Créer le site de production :

```bash
cd /home/user/espace-travail-user/notes-du-user
sudo mkdir /var/www/html/mkdocs
mkdocs build -c -d /var/www/html/mkdocs/
```

Le dossier _mkdocs_ est exploité par un serveur web _nginx_.

La mise à jour du serveur de production s'effectue ainsi :

```bash
cd /home/user/espace-travail-user/notes-du-user
sudo mkdocs build -c -d /var/www/html/mkdocs/
```

**Fin.**
