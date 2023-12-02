---
title: Bases - MkDocs
summary: Utilisation de MkDocs.
authors: Gerard Leloup
date: 2023-06-01
---

## Les bases de MkDocs

Pour la documentation complète, visiter [mkdocs.org](https://www.mkdocs.org){ target="_blank" }.

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

## Aide à l'écriture

Titres : h1 > h2 > h3 etc...

Image - options : { loading=lazy }, {:target="_blank"}, { align=left }

Image - syntaxe :

`![Logo - ...](../wp-content/uploads/2019/02/logo-virtualbox.jpg){ align=left }`

![Logo - VirtualBox](../wp-reseau-virtuel/wp-content/uploads/2019/02/logo-virtualbox.jpg){ align=left }

&nbsp;  
Image  
alignée  
à gauche.
&nbsp;  
&nbsp;

Image - légende : Paragraphe oblique et gras (Voir la feuille de style extra.css)

Image centrée : ../image.webp#center (Voir la feuille de style extra.css)

Lien - options : { target="_blank" }

Lien - syntaxe :

`[VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" }`

Exemple : [VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" }

Paragraphe - surlignage : ==texte-surligné==  (== de chaque côté du texte)

!!! note "Nota"
    "!!! note "Nota""

    Ceci est une note.

!!! Warning "Alerte"
    "!!! Warning "Alerte""

    Ceci est une alerte.

## Fichier .markdownlint.json

Fichier créé à la racine du projet.

```markdown
{
  "MD013": false, (valeur par défaut)
  "MD033": {
      "allowed_elements": [ "figure", "figcaption" ] (pour les blocs image)
  },
  "MD046": false (pour les blocs code)
}
```

## Plugin

Ajout du plugin : mkdocs-glightbox (pour les blocs image)
