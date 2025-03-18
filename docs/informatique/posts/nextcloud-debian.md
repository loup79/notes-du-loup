---
title: "Nextcloud - Sur Debian"
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

Redémarrer ensuite apache.

### Fenêtre de maintenance

Si remarque concernant l'heure de début de la fenêtre de maintenance :

```bash
sudo nano /var/www/html/nextcloud/config/config.php
```

Ajouter la ligne suivante :

```markdown
'maintenance_window_start' => 1,
```

### Fichiers - Interface Web

Cas sur une Debian :  
Le contenu du dossier *Tous les fichiers* est mis à jour manuellement par synchronisation *(FreeFileSync)* sur un dossier SFTP de la Debian.

Pour actualiser le nouveau contenu dans l'interface Web, procéder comme suit :

```bash
sudo cd /var/www/html/nextcloud/
sudo -u www-data php /var/www/html/nextcloud/occ files:scan --all
```

### Problème updater.secret

En cas de problème, il est préférable de créer une nouvelle clé en utilisant la Cde php suivante :

```bash
cd /var/www/html/nextcloud/config/

php -r '$password = trim(shell_exec("openssl rand -base64 48"));if(strlen($password) === 64) {$hash = password_hash($password, PASSWORD_DEFAULT) . "\n"; echo "Insert as \"updater.secret\": ".$hash; echo "The plaintext value is: ".$password."\n";}else{echo "Could not execute OpenSSL.\n";};'
```

Exemple de retour :

```markdown
Insert as "updater.secret": $2y$10$3eAg84tEeAtUVALFYpnvy.f6qDBOzgunN6uIFCZuma3oDWQLQfLCm

The plaintext value is: HQHlPRqnJ5Ehr9Kqwr29R+EKetH4OniPaIKatEo5mkMFGNzRtsT9zVojOfGrVxmL
```

Garder la valeur *plaintext* de côté pour une éventuelle réclamation par Nextcloud et entrer la valeur *updater.secret* dans le fichier *config.php* comme suit :

```markdown
'secret' => '$2y$10$3eAg84tEeAtUVALFYpnvy.f6qDBOzgunN6uIFCZuma3oDWQLQfLCm',
```

### Fichier nextcloud.log

Pour vider le fichier, procéder comme suit :

```bash
cd /var/www/html/nextcloud/
sudo -u www-data truncate data/nextcloud.log --size 0
```

**Fin.**
