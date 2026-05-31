---
title: MkDocs - Bases
description: Utilisation de MkDocs.
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

```markdown
mkdocs.yml    # Le fichier de configuration.
  docs/       # Le dossier contenant tous les douments
    index.md  # Le fichier de la page d'accueil.
    ...       # Autres documents markdown, images et autres fichiers.
```

## Entête d'un fichier Markdown

Exemple à respecter :

```markdown
---    
title: Titre du fichier  
description: Description sommaire.  
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
description: Description sommaire.  
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

Titres - Hiérarchie : `# = h1 -> ## = h2 -> ### = h3 etc...`

Titres - Liens internes

Par exemple, déclaration dans le document debian-ovs.md du lien interne **{#titre-4}** au titre 4 :

```markdown
### 4 - Raccordement des 2 clients Debian sur OVS {#titre-4}
```

et utilisation de celui-ci au sein d'un paragraphe quelconque du même document

```markdown
d'une ... réseau virtuelle _( [Voir le § 4](#titre-4) )_.
```

ou depuis une autre document markdown

```markdown
d'une ...  réseau virtuelle _( [Voir le § 4](debian-ovs.md#titre-4) )_.
```

### **- Images**

Image - options possibles : { loading=lazy }, { target="_blank" }, { align=left }

Image - format de la légende : Paragraphe oblique et gras _(Voir la feuille de style extra.css)_

Image - centrage : ../image.webp**#center** _(Voir la feuille de style extra.css)_

Image - syntaxe :

`![Logo - ...](image-test1.jpg){ align=left }`

![Logo - VirtualBox](image-test1.jpg){ align=left }

&nbsp;  
Image-test.jpg  
alignée  
à gauche.
&nbsp;  
&nbsp;  
&nbsp;  
&nbsp;  

ou image sans option _(Provoque une alerte markdown MD033)_

```markdown
<figure markdown>
  ![Logo de ...](../blog/images/2019/02/logo-virtualbox.jpg)
</figure>
```

ou image avec option et légende _(provoque uen alerte markdown MD033)_

```markdown
<figure markdown>
  ![Image - VBox : Logo...](../blog/images/2019/02/logo-virtualbox.jpg){ target="_blank" }
  <figcaption>VBox : Logo...</figcaption>
</figure>
```

ou image avec options _(Sans alerte markdown)_

`[![centreon-accueil-deb12.webp](image-test2.webp#center){ width="430" }](image-test2.webp)`

[![centreon-accueil-deb12.webp](image-test2.webp#center){ width="430" }](image-test2.webp)

### **- Liens**

Lien - options : { target="_blank" }

Lien - syntaxe :

`[VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" }`

Retour de l'exemple = [VirtualBox](https://www.virtualbox.org/wiki/Downloads){ target="_blank" }

Lien - URL :

```markdown
Syntaxe pour un lien actif : <https://192.168.x.y:7160>
```

<https://192.168.x.y:7160>

```markdown
Syntaxe pour un lien inactif : `https://192.168.x.y:7160`
```

`https://192.168.x.y:7160`

### - Surlignage dans blocs code

Exemple pour une entête de code _(```markdown hl_lines="2 5")_ :

```markdown hl_lines="2 5"
Client:       Podman Engine
Version:      4.3.1
API Version:  4.3.1
Go Version:   go1.19.8
Built:        Thu Jan 1 01:00:00 1970
OS/Arch:      linux/amd64
```

Note sur l'instruction hl_lines :  
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

    Ceci est une information.

Ne pas mettre entre guillements les `"!!! ..."` et indenter de 4 espaces les `Ceci est une ...`.

## Fichier .markdownlint.json

Fichier créé à la racine du projet et concernant les règles de linting du code markdown _(Qualité du code)_.

```markdown
{
  "MD013": false, # valeur par défaut
  "MD033": {
      "allowed_elements": [ "figure", "figcaption" ] # pour les blocs image
  },
  "MD046": false # pour les blocs code
}
```

## Plugin

Ajout du plugin : mkdocs-glightbox (pour les blocs image)

## Règles personnelles

Nom d'hôte par défaut : hote5  
Utilisateur par défaut : userx  
Adresse mail par défaut : `userx@mail.yz`  
relayhost = [exemple.relais.net]:587  
Expéditeur attendu par le relais SMTP (relayhost) : `userx@mail.relais.smtp.yz`

Fin des documents :  
`**Fin.**`

Flèche vers la droite :  
`Le code html &rarr;` donne &rarr;

**Fin.**
