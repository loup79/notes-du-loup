---
title: Bases - MkDocs
summary: Utilisation de MkDocs.
authors: Gerard Leloup
date: 2023-06-01
---

## Les bases de MkDocs

Pour la documentation complète, visiter [mkdocs.org](https://www.mkdocs.org).

## Commandes

`mkdocs new [dir-name]` - Pour créer un projet.  
`mkdocs serve` - Pour lancer le serveur de dévelop...  
`mkdocs build` - Pour construire la doc du site.  
`mkdocs -h` - Pour afficher l'aide.

## Schéma d'un projet

    mkdocs.yml    # Le fichier de configuration.
    docs/
        index.md  # Le fichier de la page d'accueil.
        ...       # Autres pages markdown, images et autres fichiers.

## Entête d'un fichier Markdown

Exemple à respecter :

```markdown
---
title: Titre du fichier
summary: Description sommaire.
authors: Nom de l'auteur
date: 2023-06-01
categories: 
  - "Open vSwitch + Conteneurs"
---
```

## Remarques diverses

Titres : h1 > h2 > h3 etc...

Images : { loading=lazy }, .webp#center, {:target="_blank"}, { align=left }

Format des légendes : Oblique et gras (feuille de style extra.css)

Surlignage : ==texte-surligné==  (== de chaque côté du texte)

!!! note "Nota"
    "!!! note "Nota""

    Ceci est une note

