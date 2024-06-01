---
title: Debian - Serveur Redis
summary: Application graphique informant sur le matériel installé.
authors: 
  - G.Leloup
date: 2024-03-01
categories: 
  - Debian
---

## Installation de Redis

Préambule :  
_Redis_ est un système de mise en cache qui permet d'importants gains de performance en diminuant le temps de chargement d'une page web.

_Memcached_ est un équivalent de _Redis_.

Installer le serveur :

```bash
sudo apt install redis-server phpx.y-redis
```

Vérifier son statut :

```bash
sudo systemctl status redis-server
```

Vérifier les ports utilisés par celui-ci :

```bash
sudo ss -antpl | grep redis
```

Retour :

```markdown
LISTEN  0  511   127.0.0.1:6379  0.0.0.0:*  users:(("redis-server",pid=1872,fd=6))

LISTEN  0  511       [::1]:6379     [::]:*  users:(("redis-server",pid=1872,fd=7))
```

<!-- more -->

Editer ensuite son fichier de configuration :

```bash
sudo nano /etc/redis/redis.conf
```

et vérifier ou modifier ces 3 lignes :

```markdown
bind 127.0.0.1 -::1
maxmemory 256mb 
maxmemory-policy allkeys-lru
```

Si modification, redémarrer le serveur :

```bash
sudo systemctl restart redis-server
```

_Redis_ sera ainsi utilisé en tant que _cache_.

Pour finir, ouvrir la console _Redis_ :

```bash
redis-cli
```

et lancer la Cde _ping_.

Retour normal :

```markdown
PONG
```

Utiliser la Cde _exit_ pour quitter la console _Redis_.

Commandes à connaitre :

```bash
redis-cli monitor (pour oberver le trafic)
redis-cli flushall (pour vider le cache)
```

**Fin.**
