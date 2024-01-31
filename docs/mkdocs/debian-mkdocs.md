---
title: MkDocs - Site sur Debian
summary: Installation de MkDocs sur Debian.
author: G.Leloup
date: 2023-06-01
---

## MkDocs en local sur Debian

Tuto pour installer MkDocs sur une VM Debian.

### Installation

Préalable : Python version 3 installé sur l'OS Debian.

Vérifier la version courante de Python :

```bash
python3 --version
```

et l'installation des modules Python suivants :

```bash
sudo apt list python3-pip python3-dev python3-venv python3-full
```

A défaut, installer les modules manquants à l'aide de la Cde _sudo apt install_.

Ensuite, créer un dossier qui recevra un environnement virtuel Python, ceci pour ne pas interférer par la suite avec la version de Python installé sur l'OS :

```bash
cd /home/user
mkdir -p /home/user/Documents/notes-du-user/env
cd /home/user/Documents/notes-du-user
```

et installer l'environnement Python dans le dossier _env_ :

```bash
python3 -m venv env
```

Aciver celui-ci afin de pouvoir l'exploiter :

```bash
cd env
source ./bin/activate
```

Le prompt du terminal se change en ==_(env) user@deb..._==.

et utiliser celui-ci pour installer MkDocs :

```bash
sudo pip install mkdocs
mkdocs --version
```

Les fichiers de MkDocs sont installés dans :

```bash
/usr/local/lib/python3.x/dist-packages/
```

### Création d'un projet + site Web

Créer le projet ou site de développement :

```bash
cd /home/user/espace-travail-user
mkdocs new notes-du-user
cd notes-du-user
tree
```

Retour de la Cde _tree_ :

```markdown
.
├── docs
│   └── index.md
└── mkdocs.yml
```

Créer le site de production :

```bash
cd /home/user/espace-travail-user/notes-du-user
sudo mkdir /var/www/html/mkdocs
mkdocs build -c -d /var/www/html/mkdocs/
```

Le dossier _mkdocs_ est exploité par un serveur web _nginx_.

### Ajout thème MkDocs Material

Installer le thème multilingue _mkdocs-material_ :

```bash
sudo pip3 install mkdocs-material
```

Les fichiers du thème sont installés dans :

```bash
/usr/local/lib/python3.x/dist-packages
```

!!! note "Nota"
    Lien GitHub du thème : [Material pour MkDocs](https://squidfunk.github.io/mkdocs-material/){ target="_blank" }

Pour info, 2 thèmes de base nommés _readthedocs_ et _mkdocs_ sont déjà présents dans :

```bash
/usr/local/lib/python3.x/dist-packages/mkdocs/themes/
```

Editer ensuite le fichier _mkdocs.yml_ comme suit :

```bash
cd /home/user/espace-travail-user/notes-du-user
nano mkdocs.yml
```

et entrer le contenu suivant :

```bash
site_name: Documents du user

theme:
  name: material
  language: fr
  features:
    - navigation.footer

copyright: Copyright &copy; 2023 - Cartier Jacques
```

Pour vérifier la version du thème, utiliser cette Cde :

```bash
sudo pip show mkdocs-material
```

Pour mettre à jour le thème, utiliser cette Cde :

```bash
sudo pip install --upgrade --force-reinstall mkdocs-material
```

### Serveur de développement

Pour lancer le serveur de développement, procéder ainsi :

```bash
cd /home/user/espace-travail-user/notes-du-user
mkdocs serve --dev-addr=192.168.1.x:8000
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

La mise à jour du serveur de production s'effectue ainsi :

```bash
cd /home/user/espace-travail-user/notes-du-user
sudo mkdocs build -c -d /var/www/html/mkdocs/
```

**Fin.**
