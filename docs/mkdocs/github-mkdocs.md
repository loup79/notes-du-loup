---
title: MkDocs - Site Web sur GitHub
summary: Installation de MkDocs sur GitHub.
author: G.Leloup
date: 2023-06-01
---

## MkDocs sur GitHub

Tuto pour installer MkDocs sur GitHub.

### Situation de base

MkDocs et Git installés sur une VM Debian.

Configurer le _user.name_ et le _user.email_ de git :

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

### Compte GitHub

Créer un compte GitHub en s'aidant du lien suivant :  
[Bien démarrer avec votre compte GitHub](https://docs.github.com/fr/get-started/onboarding/getting-started-with-your-github-account){ target="_blank" }

Se connecter ensuite sur celui-ci et créer un dossier en utilisant l'URL :  
[https://github.com/new](https://github.com/new){ target="_blank" }

\- Donner un nom au dossier _(Ex: notes-user)_  
\- Cocher _Public_  
\- Cocher _Add a README file_  
\- Cliquer sur le bouton _Create repository_

Le dossier _notes-user_ correspond à un nom de dépôt.

Modifier ensuite le contenu du fichier _README.md_, ceci en utilisant le langage _Markdown_ et terminer en cliquant sur le bouton _Commit changes_.

### Installation de MkDocs

L'installation sera automatisée en exploitant les possibilités de _GitHub Actions_.

_MkDocs_ et le thème _Material for Mkdocs_ seront installés à la racine du dépôt GitHub soit le dossier _notes-user_.

GitHub Actions va déployer le site MkDocs en tant que GitHub Pages, en utilisant le thème Material.

Procédure pour installer MkDocs :

1 - Dans GitHub, accéder au dépôt _notes-du-user_ et cliquer sur _Settings_.

2 - Section _Code and automation_ de la barre latérale, cliquer sur _Pages_.

!!! note "Nota"
    GitHub Pages est conçu pour héberger des pages personnelles, d'organisation ou de projet à partir d'un dépôt GitHub.

L'action suppose qu'un fichier _mkdocs.yml_ est présent dans le répertoire de premier niveau soit la branche _main_ du dépôt et que les fichiers sources _(Markdown, etc.)_ se trouvent dans le dossier _docs/_.

Pour cela, cliquer sur le bouton _Add file_ et sélectionner _Upload files_

Faire glisser le contenu du dossier _/home/user/espace-travail-user/notes-du-user/_ de la VM Debian qui contient au départ un dossier de nom _docs_ et un fichier de nom _mkdocs.yml_.

3 - Dans la zone _Build and deployment_, sous _Source_, sélectionner _Deploy from a branch_.

4 - Sous _Branch_, sélectionner _main_.

Sera utilisé un workflow GitHub Actions pour publier le site web MkDocs.

Pour cela, cliquer sur le bouton "Add file" et sélectionner Create a new file

Créer un fichier _/notes-du-user/.github/workflows/ci.yml_ contenant :

```bash
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

Cliquer ensuite sur le bouton _Commit changes..._, vérifier le contenu de la fenêtre popup et cliquer sur le bouton _Commit changes_.

Le site se construit automatiquement et si tout est OK GitHub génère une branche nommée _gh-pages_.

Ensuite déployer le site comme suit :

1 - Dans GitHub, accéder au dépôt _notes-du-user_ et cliquer sur _Settings_.

2 - Section _Code and automation_ de la barre latérale, cliquer sur _Pages_.

3 - Sous _Branch_, sélectionner _gh-pages_ et cliquer sur le bouton _Save_.

**Fin.**
