---
title: "Nextcloud - Debian"
summary: Nextcloud sur un serveur Debian.
authors: 
  - G.Leloup
date: 2024-01-02
---

## Docker - MAJ de Nextcloud

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

C'est à présent plus simple, l'outil DSM7 _Container Manager_ prévenant au niveau de l'onglet _Image_ si une mises à jour est disponible.

Il suffit de lancer la MAJ pour reconstruire le conteneur Nextcloud.

### Applications à réinstaller

```bash
apt update
apt install nano
apt install smbclient
```

### Ajout optionnel de Redis

Installer les paquets suivants :

```bash
apt update
apt install redis-server redis-tools
```

Le serveur est lancé depuis le dossier */etc/init.d*.

Compléter le fichier *config.php* de Nextcloud comme suit :

```bash
'filelocking.enabled' => true,
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' => 
  array (
    'host' => '/var/run/redis/redis-server.sock',
    'port' => 0,
  ),
```

Modifier ensuite le fichier */etc/redis/redis.conf* comme suit :

```bash
port 0
unixsocket /run/redis/redis-server.sock
unixsocketperm 770
```

Intégrer l'utilisateur *www-data* dans le groupe *redis* :

```bash
usermod -a -G redis www-data
```

Relancer Redis et Apache :

```bash
/etc/init.d/redis-server restart
/etc/init.d/apache2 restart
```

Utiliser la Cde *systemctl* si *systemd* est utilisé.

**Fin.**
