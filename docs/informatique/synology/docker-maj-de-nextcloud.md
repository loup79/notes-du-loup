# Docker - MAJ de Nextcloud

Ouvrir l'application **Docker**.

Chercher l'application **nextcloud** dans l'onglet **Registre** de Docker et télécharger la dernière version *(Latest)*.

Par précaution, vérifier sur le hub de Nextcloud que c'est bien la dernière version qui est proposée.

Une fois terminé, aller dans l'onglet **Conteneur** et stopper **nextcloud** depuis le menu **Action**.

Depuis ce même menu, **Réinitialier** le conteneur **nextcloud**.

Le conteneur sera supprimé et recréé à partir de la nouvelle image.

Redémarrer ensuite le conteneur qui ira chercher sa configuration dans **/volume1/docker/nextcloud/config**.

<mark>Fin</mark>