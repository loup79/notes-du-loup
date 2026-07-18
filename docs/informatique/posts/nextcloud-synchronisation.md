---
title: "Nextcloud - Synchronisation"
summary: Synchro de l'affichage Web avec le contenu du dossier data.
authors: 
  - G.Leloup
date: 2024-07-02
categories: 
  - Nextcloud
---

-\ Synchro avec la ligne de Cdes  
Synchroniser l'affichage Web de Nextcloud avec le contenu courant de son dossier data :

```bash
sudo -u www-data php occ files:scan --all
```

ou pour un dossier spécifique :

```bash
sudo -u www-data php occ files:scan --path=user/files/Documents/Documents-PC
```

-\ Synchro avec l'outil Crontab  
Editer le fichier des tâches crontab dédiées à l'utilisateur www-data :

```bash
sudo crontab -u www-data -e
```

et ajouter le contenu suivant pour exécuter automatiquement la tâche de synchronisation tous les lundi à 4 heures :

```markdown
0 4 * * 1 /usr/bin/php8.4 -f /var/www/html/nextcloud/occ files:scan --all > /var/www/html/nextcloud/nextcloud_rescan_debug.log 2>&1
```

ou pour un dossier spécifique

```markdown
0 4 * * 1 /usr/bin/php8.4 -f /var/www/html/nextcloud/occ files:scan --path=user/files/Documents/Documents-PC > /var/www/html/nextcloud/nextcloud_rescan_debug.log 2>&1
```

**Fin.**
