---
title: "Nextcloud - Sur Synology"
summary: Conteneur Nextcloud sur un serveur Synology.
authors: 
  - G.Leloup
date: 2023-06-02
categories: 
  - Nextcloud
---

## Conteneur Nextcloud

### MAJ de Nextcloud (2023)

Ouvrir l'application **Docker** du Synology.

Par précaution, vérifier auparavant sur le hub de Docker que c'est bien la dernière version de Nextcloud qui est proposée.

Chercher ensuite sur le Synology l'application **nextcloud** dans l'onglet **Registre** de Docker et télécharger la dernière version *(Latest)*.

Une fois terminé, aller dans l'onglet **Conteneur** et stopper **nextcloud** depuis le menu **Action**.

Depuis ce même menu, **Réinitialier** le conteneur **nextcloud**.

Le conteneur sera supprimé et recréé à partir de la nouvelle image.

Redémarrer ensuite le conteneur qui ira chercher sa configuration dans :  
**/volume1/docker/nextcloud/config**.

### MAJ de Nextcloud (2024)

<!-- more -->

C'est à présent plus simple, l'application DSM7 **Container Manager** prévenant au niveau de son onglet *Image* si une mises à jour est disponible.

Il suffit de lancer la MAJ pour reconstruire le conteneur Nextcloud.

#### Applications à réinstaller

Cdes à lancer depuis le terminal associé au conteneur :

```bash
apt update
apt install nano  # Utile pour modifier les fichiers du conteneur
apt install smbclient
```

#### Module PHP bz2

Si remarque concernant le module PHP bz2 manquant, entrer la Cde suivante :

```bash
apt install -y libbz2-dev && docker-php-ext-install bz2
```

Redémarrer ensuite le conteneur Nextcloud.

#### Fenêtre de maintenance

Si remarque concernant l'heure de début de la fenêtre de maintenance, ajouter ce qui suit dans le fichier *config.php*  de Nextcloud :

```markdown
'maintenance_window_start' => 1,
```

**Fin.**
