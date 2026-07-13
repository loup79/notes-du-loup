---
title: NAS - Synology et Redis
description: Installation de Redis pour Wordpress.
authors: 
  - G.Leloup
date: 2026-07-13
categories: 
  - NAS
---

## Serveur Redis pour WordPress

### Installation

Installation utilisant l'application Container Manager _(Docker)_ en place sur le Synology.

Ouvrir le _File Station_ du Synology et créer dans le dossier _docker_ un sous-dossier de nom _redis-wordpress_.

Ouvrir ensuite le Planificateur de tâche du Synology et :  
\-> Créer -> Tâche planifiée -> Script défini par l'utilisateur

\-> Onglet _Général_  
\-> Champ _Tâche_ -> Entrer _Redis WordPress_  
\-> Décocher la case _Activé_  
\-> Champ _Utilisateur_ -> Entrer _root_

\-> Onglet _Programmer_  
\-> Cocher _Exécuter à la date suivante_  
\-> Sélectionner _Ne pas répéter_

\-> Onglet _Paramètres de tâche_  
\-> Section _Exécuter la commande_

Entrer le contenu suivant :

```bash
docker run -d --name=Redis-WordPress \
-p 6379:6379 \
-e TZ=Europe/Paris \
-v /volume1/docker/redis-wordpress:/data \
--user 1032:100 \
--restart always \
redis
```

<!-- more -->

\-> Bouton OK

La tâche _Redis WordPress_ est créée.

1032 = UID utilisateur et 100 = GID utilisateur.

Pour trouver les valeurs correctes, exécuter le script _Mes UID et GID_ créé sur le Synology.

Sélectionner ensuite la tâche _Redis WordPress_ créée et cliquer sur le bouton _Exécuter_.

Si tout se passe bien, le serveur s'installe et démarre.

### Contrôle de fonctionnement

Se connecter en SSH sur le Synology et entrer cette Cde :

```bash
charlot@syno $ redis-cli ping
```

Retour normal :

```markdown
PONG
```

### Ajout du module PhpRedis

Le module PHP _redis.so_ est présent sur le Synology.

Pour le vérifier, entrer la Cde suivante :

```bash
charlot@syno $ ls /volume1/@appstore/PHP8.0/usr/local/lib/php80/modules/redis.so
```

Un fichier d'activation _redis.ini_ est présent dans :

```markdown
/volume1/@appstore/PHP8.0/etc/php/conf.d
```

Pour exploiter celui-ci dans DSM 7, éditer ce fichier :

```bash
charlot@syno $ sudo vi /volume1/@appstore/PHP_8.0/misc/extension_list.json
```

et entrer ce contenu sous la partie _posix_ :

```markdown
"redis": {
"enable_default": true,
"desc": "The phpredis extension provides an API for communicating with the Redis key-value store."
},
```

Pour une prise en compte du module, redémarrer _nginx_ :

```bash
charlot@syno $ sudo synosystemctl restart nginx
```

Il est maintenant possible d'activer ou de désactiver le module PHP _redis_ depuis l'application _Web Station_ dans la section _Paramètres du langage de script_.

Pour vérifier la bonne activation du module, il suffit d'exécuter le fichier _phpinfo.php_.

WordPress a besoin de ce module pour que le plugin _Redis Object Cache_ puisse dialoguer avec le serveur Redis du Synology.

**Fin.**
