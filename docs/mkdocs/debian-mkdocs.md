---
title: Debian - MkDocs
summary: Installation de MkDocs sur Debian.
authors:
    - Gerard Leloup
date: 2023-06-01
---

Tuto pour installer MkDocs sur une VM Debian 11.

### **1- Installation**
Préalable : Python version 3 installé sur la Debian.

Vérifier la version courante de Python :
```
$ python3 --version
```

Ensuite ajouter si manquant le module *pip* :
```
$ sudo apt install python3-pip
```

et utiliser celui-ci pour installer MkDocs :
```
$ sudo pip install mkdocs
$ mkdocs --version
```

Les fichiers de MkDocs sont installés dans :
```
/usr/local/lib/python3.x/dist-packages/
```

### **2 - Création d'un projet MkDocs + site Web**
Projet ou site de développement :
```
$ cd /home/user/espace-travail-user
$ mkdocs new notes-du-user
$ cd notes-du-user
$ tree
```

Retour de la Cde *tree* :
```
.
├── docs
│   └── index.md
└── mkdocs.yml
```

Site de production :
```
$ cd /home/user/espace-travail-user/notes-du-user
$ sudo mkdir /var/www/html/mkdocs
$ mkdocs build -c -d /var/www/html/mkdocs/
```

Le dossier *mkdocs* est exploité par un serveur web *nginx*.

### **3 - Installation d'un thème pour Mkdocs**
Installer le thème multilingue *mkdocs-material* :
```
$ sudo pip3 install mkdocs-material
```

Les fichiers du thème sont installés dans :
```
/usr/local/lib/python3.x/dist-packages
```

**Nota -** Lien GitHub du thème : [Material pour MkDocs](https://squidfunk.github.io/mkdocs-material/)

Pour info, 2 thèmes de base nommés *readthedocs* et *mkdocs* sont dans :
```
/usr/local/lib/python3.x/dist-packages/mkdocs/themes/
```

Editer ensuite le fichier *mkdocs.yml* comme suit :
```
$ cd /home/user/espace-travail-user/notes-du-user
$ nano mkdocs.yml
```

et entrer le contenu suivant :
```
site_name: Documents du user

theme:
  name: material
  language: fr
  features:
    - navigation.footer

copyright: Copyright &copy; 2023 - Cartier Jacques
```

Pour vérifier la version du thème, utiliser cette Cde :
```
$ sudo pip show mkdocs-material
```

Pour mettre à jour le thème, utiliser cette Cde :
```
$ sudo pip install --upgrade --force-reinstall mkdocs-material
```

#### **4 - Serveur de développement**
Pour lancer le serveur, procéder ainsi :
```
$ cd /home/user/espace-travail-user/notes-du-user
$ mkdocs serve --dev-addr=192.168.1.x:8000
```
L'IP 192.168.1.x est celle de la VM Debian 11.

Retour normal :
```
INFO     -  Building documentation...
INFO     -  Cleaning site directory
INFO     -  Documentation built in 1.20 seconds
INFO     -  [12:10:49] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO     -  [12:10:49] Serving on http://192.168.1.x:8000/
```

Touches *CTRL+C* pour fermer le serveur.

### **5 - MAJ du serveur de production**
La mise à jour s'effectue ainsi :
```
$ cd /home/user/espace-travail-user/notes-du-user
$ sudo mkdocs build -c -d /var/www/html/mkdocs/
```

**Fin**