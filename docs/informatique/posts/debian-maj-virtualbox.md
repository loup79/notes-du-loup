---
title: Debian - MAJ de VirtualBox
description: Mise à jour de VirtualBox sous Debian.
authors: 
  - G.Leloup
date: 2026-07-14
categories: 
  - Debian
---

## Mise à jour de VirtualBox

### Clé GPG

\- Sous Debian 13

Les clés GPG sur une Debian sont stockées dans le fichier _/etc/apt/trusted.gpg_ ou dans les dossiers _/etc/apt/trusted.gpg.d/_, _/etc/apt/keyrings/_ et _/usr/share/keyrings_.

Installer les clés nécessaires aux MAJ de VirtualBox :

```bash
wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc | gpg --dearmor --yes --output /usr/share/keyrings/oracle_vbox_2016.gpg
```

### Dépôt

Créer ensuite le fichier _/etc/apt/sources.list.d/virtualbox.sources_ et entrer ce contenu :

```markdown
Types: deb
URIs: http://download.virtualbox.org/virtualbox/debian/
Suites: trixie
Components: contrib
Signed-By: /usr/share/keyrings/oracle_vbox_2016.gpg
```

### Pack d'extension

<!-- more -->

Cde pour lister le pack d'extension utilisé :

```bash
vboxmanage list extpacks
```

Pour installer un nouveau pack, télécharger celui-ci sur le site Web de VirtualBox puis se rendre dans le dossier _/home/user/Téléchargements_.

Enfin, lancer la Cde suivante :

```bash
sudo vboxmanage extpack install --replace Oracle_VirtualBox_Extension_Pack-x.y.z.vbox-extpack
```

**Fin.**
