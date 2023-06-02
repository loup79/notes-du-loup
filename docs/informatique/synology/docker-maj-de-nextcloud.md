---
title: Docker - MAJ de Nextcloud
summary: Mise à jour du conteneur docker Nextcloud sur un serveur Synology.
authors: Gerard Leloup
date: 2023-06-02
---

# Docker - MAJ de Nextcloud

Ouvrir l'application **Docker** du Synology.

Par précaution, vérifier auparavant sur le hub de Docker que c'est bien la dernière version de Nextcloud qui est proposée.

Chercher ensuite sur le Synology l'application **nextcloud** dans l'onglet **Registre** de Docker et télécharger la dernière version *(Latest)*.

Une fois terminé, aller dans l'onglet **Conteneur** et stopper **nextcloud** depuis le menu **Action**.

Depuis ce même menu, **Réinitialier** le conteneur **nextcloud**.

Le conteneur sera supprimé et recréé à partir de la nouvelle image.

Redémarrer ensuite le conteneur qui ira chercher sa configuration dans :  
**/volume1/docker/nextcloud/config**.

**Fin**