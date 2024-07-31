---
title: MkDocs - Bases
summary: Utilisation de MkDocs.
author: G.Leloup
date: 2023-06-01
---

## Les bases de MkDocs

Pour la doc complète, visiter [mkdocs.org](https://www.mkdocs.org){ target="_blank" }.  
Pour la doc du thème, visiter [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/){ target="_blank" }.

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

ou

```markdown
---    
title: Titre du fichier  
summary: Description sommaire.  
authors: 
  - Nom de l'auteur  
date: 2023-06-01  
categories:   
  - "Open vSwitch + Conteneurs"  
---
```

si l'on veut exploiter le fichier _blog/.authors.yml_ dont le contenu peut ressembler à ceci :

```markdown
authors:
  Nom de l'auteur:
    name: Pseudonyme
    description: Divers
    avatar: https://xxx/blog/avatar.png
```

## Aide à l'écriture

### **- Titres**

Titres : h1 > h2 > h3 etc...

Titres - Liens internes :

Ajout, par exemple, de **{#titre-4}** au titre 4

```bash
### 4 - Raccordement des 2 clients Debian sur OVS {#titre-4}
```

et création d'un lien au sein d'un paragraphe quelconque

```bash
d'une ... réseau virtuelle _( [Voir le § 4](#titre-4) )_.
```

ou création d'un lien depuis une autre page Web

```bash
d'une ...  réseau virtuelle _( [Voir le § 4](post-x.md#titre-4) )_.
```

### **- Images**

Image - options : { loading=lazy }, {:target="_blank"}, { align=left }

Image - légende : Paragraphe oblique et gras _(Voir la feuille de style extra.css)_

Image - centrage : ../image.webp**#center** _(Voir la feuille de style extra.css)_

Image - syntaxe :

`![Logo - ...](../wp-content/uploads/2019/02/logo-virtualbox.jpg){ align=left }`

![Logo - VirtualBox](../blog/images/2019/02/logo-virtualbox.jpg){ align=left }

&nbsp;  
Image  
alignée  
à gauche.
&nbsp;  
&nbsp;  
&nbsp;  
&nbsp;  

ou image sans option _(Alerte markdown MD033)_

```bash
<figure markdown>
  ![Logo de ...](../blog/images/2019/02/logo-virtualbox.jpg)
</figure>
```

ou image avec option et légende _(Alerte markdown MD033)_

```bash
<figure markdown>
  ![Image - VBox : Logo...](../blog/images/2019/02/logo-virtualbox.jpg){ width="xxx" }
  <figcaption>VBox : Logo...</figcaption>
</figure>
```

### **- Liens**

Lien - options : { target="_blank" }

Lien - syntaxe :

`[VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" }`

Exemple : [VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" }

Lien - URL http et https :

```bash
Exemple de lien actif : <http://192.168.x.y:7160>
```

```bash
Exemple de lien inactif : `http://192.168.x.y:7160`
```

### - Cdes/Scripts et retours

Cdes/Scripts _( code ```bash, yaml, json, etc... )_ :

```bash
sudo nano /etc/systemd/system/networknamespace.service
```

Retours _(code ```markdown hl_lines="2 5" )_ :

```markdown hl_lines="2 5"
Client:       Podman Engine
Version:      4.3.1
API Version:  4.3.1
Go Version:   go1.19.8
Built:        Thu Jan 1 01:00:00 1970
OS/Arch:      linux/amd64
```

L'instruction :  
hl_lines="4" permet de surligner la ligne 4.  
hl_lines="2 5" permet de surligner les lignes 2 et 5.  
hl_lines="3-7" permet de surligner les lignes 3 à 7.

### **- Divers**

Texte - centrage : `<center>---------- Fin ----------</center>` _(Alerte markdown MD033)_

Paragraphe - surlignage : ==texte-surligné==  (== de chaque côté du texte)

Extraits - longueur : Inclure le séparateur `<!-- more -->` dans les articles.

### **- Fenêtres spéciales**

!!! note "Nota"
    "!!! note "Nota""

    Ceci est une note.

!!! Warning "Alerte"
    "!!! Warning "Alerte""

    Ceci est une alerte.

!!! Info "Mémento 6.1 en cours de construction"
    "!!! Info "Mémento 6.1 en cours de construction""

## Fichier .markdownlint.json

Fichier créé à la racine du projet.

```bash
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
