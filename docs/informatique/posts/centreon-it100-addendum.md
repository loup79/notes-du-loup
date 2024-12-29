---
title: "Centreon - Addendum"
summary: Centreon IT 100.
authors: 
  - G.Leloup
date: 2024-12-29
categories: 
  - Centreon
---

<figure markdown>
  ![Capture - Centreon : Tableau de bord](../images/2024/11/astuce.jpg){ width="430" }
</figure>

## Centreon IT-100 - Addendum

### Désinstallation complète

Tout sera supprimé, y compris la configuration des hôtes et des services, le but étant de tout reconstruire suite à un incident bloquant sur Centreon tel un problème de mise à jour, etc...

Si impossibilité de dissocier la licence comme indiqué en _Etape 1_, importer dans ce cas une **sauvegarde** de la VM Centreon et effectuer les opérations décrites ci-dessous sur celle-ci.

!!! Note "Avantages d'une sauvegarde"
    1 - Elle peut aider à ne pas avoir à contacter le support de Centreon pour régler un problème de licence.  
    2 - Elle peut aussi, selon le cas de figure, éviter d'avoir à tout désinstaller et reconstruire.

#### _- Etape 1_

!!! Warning "Alerte"
    Commencer par dissocier la licence IT-100 afin de pouvoir associer celle-ci sur une nouvelle installation.

Dissocier la licence :  
-> Administration -> Extensions -> Gestionnaire  
-> Bouton _Voir la licence_  
-> Bouton _Dissocier votre plate-forme_

Ensuite, arrêter et désactiver le service _centreon_ :

```bash
sudo systemctl stop centreon
sudo systemctl disable centreon
```

Puis, supprimer avec Adminer ou phpMyAdmin les Bdd _centreon_ et _centreon_storage_.

#### _- Etape 2_

Supprimer l'ensemble des paquets :

```bash
sudo apt remove centreon*
```

#### _- Etape 3_

Supprimer les fichiers du dossier /etc :

```bash
sudo rm -r /etc/centreon
sudo rm -r /etc/centreon-broker
sudo rm -r /etc/centreon-engine
sudo rm -r /etc/centreon-gorgone
```

<!-- more -->

#### _- Etape 4_

Supprimer les fichiers du dossier /var/lib :

```bash
sudo rm -r /var/lib/centreon
sudo rm -r /var/lib/centreon-broker
sudo rm -r /var/lib/centreon-engine
sudo rm -r /var/lib/centreon-gorgone
```

#### _- Etape 5_

Supprimer les fichiers du dossier /usr :

```bash
sudo rm -r /usr/share/centreon
sudo rm -r /usr/share/centreon-broker
sudo rm -r /usr/share/centreon-packs
sudo rm -r /usr/lib/centreon
```

#### _- Etape 6_

Supprimer les fichiers du dossier /var/log :

```bash
sudo rm -r /var/log/centreon
sudo rm -r /var/log/centreon-broker
sudo rm -r /var/log/centreon-engine
sudo rm -r /var/log/centreon-gorgone
```

#### _- Etape finale_

Supprimer les fichiers du dossier /var/cache :

```bash
sudo rm -r /var/cache/centreon
sudo rm -r /var/cache/centreon-gorgone
```

Il est maintenant possible de réinstaller complètement Centreon IT-100.

**Fin.**
