---
title: Debian - MkDocs
---

### **1- Installation**
Préalable : Python version 3 installé sur la Debian.

Vérifier la version :
```
$ python3 --version
```

Ensuite ajouter le module *pip* de Python :
```
$ sudo apt install python3-pip
```

et utiliser celui-ci pour installer MkDocs :
```
$ sudo pip install mkdocs
$ mkdocs --version
```

Les fichiers MkDocs ont été installés dans :
```
/usr/local/lib/python3.x/dist-packages/
```

### **2 - Création d'un projet + site**
Projet ou site de développement :
```
$ cd /home/user/espace-travail-user
$ mkdocs new mkdocs-user
$ cd mkdocs-user
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
$ sudo mkdir /var/www/html/mkdocs
$ mkdocs build -c -d /var/www/html/mkdocs/
```

### **3 - Installation d'un thème**
Thème multilingue *mkdocs-material* :
```
$ sudo pip3 install mkdocs-material
```

L'installation s'est faite dans :
```
/usr/local/lib/python3.x/dist-packages
```

Lien du thème : [Material pour MkDocs](https://squidfunk.github.io/mkdocs-material/)

Pour info, d'autres thèmes de base nommés *readthedocs* et *mkdocs* se trouvent dans :
```
/usr/local/lib/python3.x/dist-packages/mkdocs/themes/
```

Modifier ensuite le fichier *mkdocs.yml* comme suit :
```
$ cd mkdocs-user
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

copyright: Copyright &copy; 2023 - Leloup G&eacute;rard
```

Vérifier la version du thème :
```
$ sudo pip show mkdocs-material
```

Mise à jour du thème :
```
$ sudo pip install --upgrade --force-reinstall mkdocs-material
```

#### **4 - Serveur de développement**
lancement du serveur :
```
$ cd mkdocs-user
$ mkdocs serve --dev-addr=192.168.1.x:8000
```

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
$ sudo mkdocs build -c -d /var/www/html/mkdocs/
```