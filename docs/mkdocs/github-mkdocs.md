---
title: MkDocs - Site Web sur GitHub
summary: Installation de MkDocs sur GitHub.
author: G.Leloup
date: 2023-06-01
---

## MkDocs sur GitHub

Tuto pour installer MkDocs sur GitHub.

### Situation de base

MkDocs _(serveur de production)_, Git et VS Code Server installés dans un conteneur _(CTN)_ Qnap supportant Debian.

MkDocs est situé dans le dossier /var/www/html/mkdocs/.

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

Cette branche contiendra ==la documentation de MkDocs== qui sera mise à jour depuis le _VS Code Server_ installé sur le CTN Debian.

Finir en modifant le contenu du fichier _README.md_, ceci en utilisant le langage _Markdown_ et terminer en cliquant sur le bouton _Commit changes_.

### Authentification par jeton

Le jeton _(token)_  remplace complètement l’authentification SSH, donc pas besoin d'un agent SSH.

Cde à lancer depuis le CTN Debian pour lire les URL d'accès à GitHub :

```bash
git remote -v
```

Exemple de retour :

```markdown
origin  https://git@github.com/user-git/nom-depot.git (fetch)

origin  https://git@github.com/user-git/nom-depot.git (push)
```

Se procurer un jeton depuis le dépôt GitHub, menu _Settings_ :  
-> Developer Settings  
-> Personal access tokens  
-> Tokens (classic) .

Déclarer ensuite le jeton dans l'espace de travail de VS Code Server comme suit :

```bash
git remote set-url origin https://valeur-du-token@github.com/user-git/nom-depot-git
```

Ainsi la fonction _push_ de VS Code Server vers GitHub fonctionnera sans demande de MDP.

Relancer cette cde pour vérifier la prise en compte :

```bash
git remote -v
```

Retour à observer :

```markdown
origin  https://ghp_Y3PD41f...@github.com/user-git/nom-depot-git (fetch)

origin  https://ghp_Y3PD41f...@github.com/user-git/nom-depot-git (push)
```

Le jeton _(token)_ a une durée de vie qui peut être renouvelée.

### Renouvellement du jeton

Se connecter sur le dépôt GitHub, menu _Settings_ :  
-> Developer Settings  
-> Personal access tokens  
-> Tokens (classic)  
-> Clic sur le lien du jeton existant  
-> Bouton Regenerate Token.

Enregister la valeur du nouveau token et lancer la Cde suivante depuis le CTN Debian :

```bash
git remote set-url origin https://valeur-du-token@github.com/user-git/nom-depot-git
```

Vérifier le résultat comme suit :

```bash
git remote -v
```

### Option agent SSH

Git, si utilisation de la Cde _git@github.com:..._, a besoin d’une clé SSH privée pour prouver à GitHub son identité.

L’agent SSH du CTN Debian :

- Chargera la clé privée SSH en mémoire

- Répondra aux demandes d’authentification SSH

- Permettra à Git de pousser vers GitHub _( git@github.com:... )_

- Permettra le déploiement du serveur de production

- Evitera d'avoir à taper le MDP ou la passphrase à chaque opération

- Protègera la clé privée en ne la divulguant jamais

Clés ssh créées dans le /home/user/.ssh du conteneur.  
Clé privée ajoutée à l'agent ssh, Cde _ssh-add_.  
Clé privée incluant une phrase secrète _(passphrase)_.  
Clé publique ajoutée sur le compte GitHub _(Settings)_.

Lignes ajoutées dans le fichier _/home/user/.bashrc_ :

```markdown
# Gestion clé SSH pour GitHub
eval `ssh-agent`
ssh-add /home/user/.ssh/nom-cle-privee-ssh
```

La _passphrase_ est demandée à chaque ouverture de session SSH.

Si OK, la Cde _ssh-add -l_ doit montrer ceci :

```markdown
256 SHA256:DyVA8KvbmJdRqp... name-user-mail@xxx.com (ED25519)
```

Pour tester une connexion sur GitHub, procéder comme suit :

```bash
ssh -vT git@gitub.com
```

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

Faire glisser le contenu du dossier _/home/user/Documents/notes-user/_ du CTN Debian qui contient au départ un dossier de nom _docs_ et un fichier de nom _mkdocs.yml_.

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

[Lien](https://squidfunk.github.io/mkdocs-material/publishing-your-site/){ target="_blank" } vers la référence du code entré ci-dessus.

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

### Cdes Git depuis le CTN Debian

Se rendre au préalable dans le dossier _/home/user/Documents/notes-user/_.

```bash
git branch
```

Retour :

```markdown
  gh-pages
* main
```

```bash
git status
```

Retour :

```markdown
Sur la branche main
Votre branche est à jour avec 'origin/main'.

rien à valider, la copie de travail est propre
```

```bash
git switch gh-pages
```

Retour possible :

```markdown
Basculement sur la branche 'gh-pages'
Votre branche et 'origin/gh-pages' ont divergé,
et ont 1 et 1 commits différents chacune respectivement.
  (utilisez "git pull" pour fusionner la branche distante dans la vôtre)
```

Pour corriger la divergence indiquée ci-dessus :

```bash
git pull origin gh-pages --rebase
```

### Copie du site Web sur le QNAP

Générer une copie du site comme suit :

```bash
cd /home/user/Documents/notes-user/
sudo mkdocs build -c -d /var/www/html/mkdocs/
```

### Thème Material for MkDocs

Mise à jour du thème pour le site sur GitHub :

```bash
cd /home/user/Documents/notes-user/
pip3 install --upgrade --force-reinstall mkdocs-material 
```

Mise à jour du thème pour le site sur le QNAP :

Les fichiers du thème se trouvent dans _/home/user/.local/lib/python3.x/site-packages/material_.

```bash
cd /home/user/.local/lib/python3.x/site-packages/material/
pip3 show mkdocs-material
sudo pip3 install --upgrade --force-reinstall mkdocs-material
pip3 show mkdocs-material 
```

**Fin.**
