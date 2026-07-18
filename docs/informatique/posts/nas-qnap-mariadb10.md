---
title: NAS - Qnap et MariaDB 10
description: Configuration de Mariadb 10 sur le NAS Qnap.
authors: 
  - G.Leloup
date: 2026-07-13
categories: 
  - NAS
---

## notes utiles

Accès aux Bdd du serveur Mariadb :

```bash
sudo ls /share/CACHEDEV1_DATA/.mariadb10/data/
```

Connexion sur le serveur :

```bash
sudo /mnt/ext/opt/mariadb/bin/mysql -u root -p
```

Fichier de configuration exploité :

```bash
sudo cat /etc/my.cnf
```

Port et socket utilisés :

```markdown
Port = xxxx
Socket = /var/run/mariadb10.sock  
```

Dossier d'installation de Mariadb 10 :

```bash
sudo ls /mnt/ext/opt/mariadb/
```

Cdes pouvant être utilisées une fois connecté :

```bash
SELECT @@datadir;  
SHOW GLOBAL VARIABLES LIKE 'PORT';  
UPDATE user SET password=PASSWORD('motdepasseroot') WHERE user="root";  
GRANT ALL PRIVILEGES ON *.* TO root@localhost  IDENTIFIED BY 'motdepasseroot';
```

Lecture des fichiers log :

```bash
sudo cat /var/log/mariadb10/mariadb.err
sudo cat /var/log/nc-mariadb.err
```

Créer une section [mysqli] dans le fichier /php.ini du Qnap et entrer ce contenu :

```markdown
[mysqli]
mysqli.default_port = xxxx
mysqli.default_socket = "/var/run/mariadb10.sock"
```

**Fin.**
