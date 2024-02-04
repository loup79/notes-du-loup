---
title: "Nextcloud sur Debian"
summary: Nextcloud sur un serveur Debian.
authors: 
  - G.Leloup
date: 2024-01-02
categories: 
  - Nextcloud
---

## Nextcloud

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

<!-- more -->

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

### Module PHP bz2

Si remarque concernant le module PHP bz2 manquant, entrer les Cdes suivantes :

```bash
apt install -y libbz2-dev
apt install phpx.y-bz2
```

Redémarrer ensuite le conteneur Nextcloud.

### Fenêtre de maintenance

Si remarque concernant l'heure de début de la fenêtre de maintenance :

```bash
sudo nano /var/www/html/nextcloud/config/config.php
```

Ajouter la ligne suivante :

```markdown
'maintenance_window_start' => 1,
```

**Fin.**
