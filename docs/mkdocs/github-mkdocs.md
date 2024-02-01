---
title: MkDocs - Site Web sur GitHub
summary: Installation de MkDocs sur GitHub.
author: G.Leloup
date: 2023-06-01
---

## MkDocs sur GitHub

Tuto pour installer MkDocs sur GitHub.

### Situation de base

MkDocs, Git et VS Code Server installés sur une VM Debian.

Configurer le _user.name_ et le _user.email_ de git ainsi :

```bash
git config --global user.name "Jacques Cartier"
git config --global user.email "j.cartier@gmail.com"
```

Vérifier le résulat de la configuration :

```bash
git config --list
```

Retour normal :

```markdown
user.name=Jacques Cartier
user.email=j.cartier@gmail.com
```

### Création du compte GitHub

Créer ensuite un compte GitHub en s'aidant du lien suivant :  
[Bien démarrer avec votre compte GitHub](https://docs.github.com/fr/get-started/onboarding/getting-started-with-your-github-account){ target="_blank" }

Se connecter ensuite sur celui-ci et créer un dossier en utilisant l'URL : [https://github.com/new](https://github.com/new){ target="_blank" }

\- Donner un nom au dossier _(Ex: notes-user)_  
\- Cocher _Public_  
\- Cocher _Add a README file_  
\- Cliquer sur le bouton _Create repository_

Le dossier/dépôt GitHub créé avec le nom _notes-user_ inclus une branche de base appelée _main_.

Cette branche contiendra ==la documentation de MkDocs== qui sera mise à jour automatiquement depuis le _VS Code Server_ installé sur la VM Debian.

Finir en modifant le contenu du fichier _README.md_, ceci en utilisant le langage _Markdown_ et terminer en cliquant sur le bouton _Commit changes_.

### Création du site Web MkDocs

La création sera automatisée en exploitant les possibilités de _GitHub Pages_ qui est un service d’hébergement de site Web statique.

Le site Web de _MkDocs_ et le thème _Material for Mkdocs_ seront installlés dans une branche appelée _gh-pages_.

!!! note ""
    GitHub Pages est conçu pour héberger des pages personnelles ou de projet à partir d'un dépôt GitHub.

Création de la branche ==_gh-pages_== :

1 - Dans GitHub, accéder au dépôt _notes-user_ et cliquer sur _Settings_.

2 - Section _Code and automation_ de la barre latérale, cliquer sur _Pages_.

La suite suppose qu'un fichier _mkdocs.yml_ soit présent dans le dossier de premier niveau du dépôt GitHub _(branche main)_ ainsi que la _documentation Mkdocs_ dans le dossier _docs/_.

Pour cela, ajouter ceux-ci en cliquant sur le bouton _Add file_ et sélectionner _Upload files_

Faire glisser le contenu du dossier _/home/user/Documents/notes-user/_ de la VM Debian qui contient au départ un dossier de nom _docs_ et un fichier de nom _mkdocs.yml_.

3 - Dans la zone _Build and deployment_, sous _Source_, sélectionner _Deploy from a branch_.

4 - Sous _Branch_, sélectionner _main_.

Un workflow GitHub _(processus automatisé)_ sera utilisé pour publier le site web MkDocs.

Pour cela, cliquer sur le bouton _Add file_ et sélectionner _Create a new file_.

Créer un fichier _/notes-user/.github/workflows/ci.yml_ contenant :

```markdown
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

[Référence du code ci-dessus](https://squidfunk.github.io/mkdocs-material/publishing-your-site/){ target="_blank" }

-> Cliquer ensuite sur le bouton _Commit changes..._  
-> Vérifier le contenu de la fenêtre popup  
-> Cliquer sur le bouton _Commit changes_

Le site se construit automatiquement et GitHub génère une branche nommée ==_gh-pages_==.

### Déploiement du site Web

Déployer ou publier le site comme suit :

1 - Dans GitHub, accéder au dépôt _notes-user_ et cliquer sur _Settings_.

2 - Section _Code and automation_ de la barre latérale, cliquer sur _Pages_.

3 - Sous _Branch_, sélectionner _gh-pages_.

4 - Cliquer sur le bouton _Save_.

### Cdes Git depuis la VM Debian

Se rendre au préalable dans le dossier _/home/user/Documents/notes-user/_.

```bash
git branch
```

Retour :

```markdown
  gh-pages
* main
```

**Fin.**
