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

### **- Titres**

Titres : h1 > h2 > h3 etc...

Titres - Liens internes :

Ajout, par exemple, de **{#titre-4}** au titre 4

```markdown
### 4 - Raccordement des 2 clients Debian sur OVS {#titre-4}
```

et création d'un lien au sein d'un paragraphe quelconque

```markdown
... d'une interface réseau virtuelle TAP _( [Voir le § 4](#titre-4) )_.
```

### **- Images**

Image - options : { loading=lazy }, {:target="_blank"}, { align=left }

Image - légende : Paragraphe oblique et gras _(Voir la feuille de style extra.css)_

Image - centrage : ../image.webp**#center** _(Voir la feuille de style extra.css)_

Image - syntaxe :

`![Logo - ...](../wp-content/uploads/2019/02/logo-virtualbox.jpg){ align=left }`

![Logo - VirtualBox](../wp-reseau-virtuel/wp-content/uploads/2019/02/logo-virtualbox.jpg){ align=left }

&nbsp;  
Image  
alignée  
à gauche.
&nbsp;  
&nbsp;

ou image sans option _(Alerte markdown MD033)_

```markdown
<figure markdown>
  ![Logo de ...](../wp-reseau-virtuel/wp-content/uploads/2019/02/logo-virtualbox.jpg)
</figure>
```

ou image avec option et légende _(Alerte markdown MD033)_

```markdown
<figure markdown>
  ![Image - VBox : Logo...](../wp-reseau-virtuel/wp-content/uploads/2019/02/logo-virtualbox.jpg){ width="xxx" }
  <figcaption>VBox : Logo...</figcaption>
</figure>
```

### **- Liens**

Lien - options : { target="_blank" }

Lien - syntaxe :

`[VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" }`

Exemple : [VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" }

Lien - URL http et https :

```markdown
Exemple de lien actif : <http://192.168.x.y:7160>
```

```markdown
Exemple de lien inactif : `http://192.168.x.y:7160`
```

### - Cdes/Scripts et retours

Cdes/Scripts _( code ```bash )_ :

```bash
sudo nano /etc/systemd/system/networknamespace.service
```

Retours _(code ```markdown )_ :

```markdown
Client:       Podman Engine
Version:      4.3.1
API Version:  4.3.1
Go Version:   go1.19.8
Built:        Thu Jan 1 01:00:00 1970
OS/Arch:      linux/amd64
```

### **- Divers**

Paragraphe - surlignage : ==texte-surligné==  (== de chaque côté du texte)

Extraits - longueur : Inclure le séparateur `<!-- more -->` dans les articles.

### **- Fenêtres spéciales**

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
