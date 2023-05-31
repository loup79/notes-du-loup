---
title: GitHub - MkDocs
---

### 1 - Installation
Au préalable, créer un compte GitHub.

Se connecter sur celui-ci et créer un nouveau dossier en utilisant l'URL :  
[https://github.com/new](https://github.com/new)

- Donner un nom au dossier
- Cocher *Public*
- Cocher *Add a README file*
- Cliquer sur le bouton *Create repository*

Modifier ensuite le contenu du fichier *README.md*, ceci en utilisant le langage *Markdown* et terminer en cliquant sur le bouton *Commit changes*.

Automatiser l'installation de MkDocs en exploitant les possibilités de GitHub Actions.

1 - S'assurer que la branche à utiliser comme source de publication existe déjà dans le dépôt (Ex: /notes-du-loup).

2 - Dans GitHub, accéder au dépôt du site (Ex: notes-du-loup) et cliquer sur Paramètres.

3 - Dans la section « Code et automatisation » de la barre latérale, cliquer sur Pages

4 - Sous Build and deployment, sous Source, sélectionner Deploy from a branch.

5 - Sous Branch 
A priori, laisser à None soit *main* par défaut.

En priorité, vérifier le statut de GitHub Pages en suivant ce menu :

Settings > Code and automation > Pages

GitHub Pages est conçu pour héberger des pages personnelles, d'organisation ou de projet à partir d'un dépôt GitHub.

Vous pouvez configurer votre site GitHub Pages à publier quand des changements sont poussés vers une branche spécifique, ou vous pouvez écrire un workflow GitHub Actions pour publier votre site.

Vérifier que Source = Deploy from a branch

Vérifier que Branch = 

Pour cela, créer le fichier



Pour cela, cliquer sur le bouton "Add file" et sélectionner Create a new file

créer le fichier suivant :  
/notes-du-loup/.github/workflows/ci.yml

et entrer ce contenu :

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
          python-version: 3.x
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

Cliquer ensuite sur le bouton Commit changes..., vérifier le contenu de la fenêtre popup et cliquer sur le bouton Commit changes

Le site devrait se construire automatiquement