---
title: GitHub - MkDocs
summary: Installation de MkDocs sur GitHub.
authors: Gerard Leloup
date: 2023-06-01
---

# MkDocs sur GitHub

Tuto pour installer MkDocs sur GitHub.

### **1 - Situation de base**  
MkDocs et Git installés sur une VM Debian 11.

Configurer le *user.name* et le *user.email* de git :  
```
$ git config --global user.name "Jacques Cartier"
$ git config --global user.email "j.cartier@gmail.com"
```
Vérifier le résulat de la configuration :
```
$ git config --list
```
Retour normal :
```
user.name=Jacques Cartier
user.email=j.cartier@gmail.com
```
### **2 - Installation du thème Material pour MkDocs**
Au préalable, créer un compte GitHub en s'aidant du lien suivant :  
[Bien démarrer avec votre compte GitHub](https://docs.github.com/fr/get-started/onboarding/getting-started-with-your-github-account)

Se connecter ensuite sur celui-ci et créer un dossier en utilisant l'URL :  
[https://github.com/new](https://github.com/new)

- Donner un nom au dossier (Ex: notes-du-user)
- Cocher *Public*
- Cocher *Add a README file*
- Cliquer sur le bouton *Create repository*

Le dossier *notes-du-user* correspond à un nom de dépôt.

Modifier ensuite le contenu du fichier *README.md*, ceci en utilisant le langage *Markdown* et terminer en cliquant sur le bouton *Commit changes*.

Automatiser l'installation de MkDocs en exploitant les possibilités de GitHub Actions.

GitHub Actions va déployer le site MkDocs en tant que GitHub Pages, en utilisant le thème Material.

Procédure pour installer MkDocs :

1 - Dans GitHub, accéder au dépôt *notes-du-loup* et cliquer sur *Settings*.

2 - Section *Code and automation* de la barre latérale, cliquer sur *Pages*.

**Nota :** GitHub Pages est conçu pour héberger des pages personnelles, d'organisation ou de projet à partir d'un dépôt GitHub.

L'action suppose qu'un fichier *mkdocs.yml* est présent dans le répertoire de premier niveau soit la branche *main* du dépôt et que les fichiers sources (Markdown, etc.) se trouvent dans le dossier docs/.

Pour cela, cliquer sur le bouton *Add file* et sélectionner *Upload files*

Faire glisser le contenu du dossier /home/user/espace-travail-user/notes-du-user/ de la VM Debian qui contient au départ un dossier de nom docs et un fichier de nom mkdocs.yml.

3 - Dans la zone *Build and deployment*, sous *Source*, sélectionner *Deploy from a branch*.

4 - Sous *Branch*, sélectionner *main*.

Sera utilisé un workflow GitHub Actions pour publier le site web MkDocs.

Pour cela, cliquer sur le bouton "Add file" et sélectionner Create a new file

Créer un fichier */notes-du-loup/.github/workflows/ci.yml* contenant :

```
name: ci 
on:
  push:
    branches:
      - master 
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: 3.x # Ex: python-version: 3.9
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV 
      - uses: actions/cache@v3
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-
      - run: pip install mkdocs-material 
      - run: mkdocs gh-deploy --force

```

[Référence du code ci-dessus](https://squidfunk.github.io/mkdocs-material/publishing-your-site/)


Cliquer ensuite sur le bouton *Commit changes...*, vérifier le contenu de la fenêtre popup et cliquer sur le bouton *Commit changes*.

Le site se construit automatiquement et si tout est OK GitHub génère une branche nommée *gh-pages*.

Ensuite déployer le site comme suit :

1 - Dans GitHub, accéder au dépôt *notes-du-loup* et cliquer sur *Settings*.

2 - Section *Code and automation* de la barre latérale, cliquer sur *Pages*.

3 - Sous *Branch*, sélectionner *gh-pages* et cliquer sur le bouton *Save*.