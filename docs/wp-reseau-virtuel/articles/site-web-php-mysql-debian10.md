---
title: "LAMP HTTPS CMS / Debian 10 : 1/2"
date: "2022-04-21"
categories: 
  - "archives"
---

## Mémento 8.1 - Apache PHP MySQL

Le service LAMP sera installé sur la VM srvdmz.

Il permettra de créer un site Web reposant sur :  
Linux + Apache + MySQL + PHP _(LAMP)_

### 1 - Préambule

Outils réutilisés :  
\- Apache pour diffuser les pages web.  
\- PHP pour générer les pages web dynamiques.  
\- MySQL/MariaDB pour stocker le contenu des pages.

Nouveautés :  
\- OpenSSL pour sécuriser le protocole HTTP _(HTTP S)_.  
\- Adminer pour gérer la base de données MySQL.  
\- WordPress pour créer facilement un site web complet.

#### _1.1 - Configuration du fichier DNS hosts_

FQDN _(Fully Qualified Domain Name)_ = Nom de domaine pleinement qualifié.

Créez celui de srvdmz en éditant son fichier DNS hosts :

\[srvdmz@srvdmz:~$\] sudo nano /etc/hosts

et en le modifiant comme ci-dessous :

127.0.0.1    localhost
127.0.1.1    srvdmz.loupvirtuel.fr srvdmz

# IP de srvdmz  +  FQDN  +  nom d'hôte  +  domaine
192.168.4.2      srvdmz.loupvirtuel.fr      srvdmz      loupvirtuel.fr      

Ne touchez pas au bloc de lignes concernant l'IPv6.

Testez la prise en compte du FQDN :

\[srvdmz@srvdmz:~$\] hostname --fqdn

Retour :

```
srvdmz.loupvirtuel.fr
```

Apache aura besoin du FQDN de srvdmz pour s'installer sans erreur.

### 2 - Installation et configuration du service LAMP

#### _2.1 - Mise à jour du système Debian_

\[srvdmz@srvdmz:~$\] sudo apt update
\[srvdmz@srvdmz:~$\] sudo apt full-upgrade
\[srvdmz@srvdmz:~$\] sudo reboot
\[srvdmz@srvdmz:~$\] cat /etc/debian\_version

La dernière Cde doit retourner le numéro de la version Debian courante.

#### _2.2 - Apache_

Installez le paquet apache2 :

\[srvdmz@srvdmz:~$\] sudo apt install apache2

et contrôlez le numéro de la version installée :

\[srvdmz@srvdmz:~$\] sudo apache2 -v 

Retour :

```
Server version: Apache/2.4.38 (Debian)
Server built:   2019-10-15T19:53:42
```

Puis vérifiez le démarrage automatique du serveur :

\[srvdmz@srvdmz:~$\] sudo systemctl status apache2  

Retour :

```
● apache2.service - The Apache HTTP Server
   Loaded: loaded (/lib/systemd/system/apache2.se...
   Active: active (running) since Sat 2019-11... ago
     Docs: https://httpd.apache.org/docs/2.4/
 Main PID: 1860 (apache2)
    Tasks: 55 (limit: 545)
   Memory: 4.7M
   CGroup: /system.slice/apache2.service
           ├─1860 /usr/sbin/apache2 -k start
           ├─1862 /usr/sbin/apache2 -k start
           └─1863 /usr/sbin/apache2 -k start

nov. 23 17:56:30 srvdmz systemd[1]: Starting The ...
nov. 23 17:56:30 srvdmz systemd[1]: Started The Ap..
```

et la bonne syntaxe de ses fichiers de configuration :

\[srvdmz@srvdmz:~$\] sudo apache2ctl -t

Retour :

```
Syntax OK
```

Enfin, depuis la VM Debian10-vm1, testez l'URL :  
http://192.168.4.2

[![Capture - Apache : Page d'accueil index.html](/wp-content/uploads/2019/11/srvdmz_apache_accueil-430x303.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/11/srvdmz_apache_accueil.jpg)

Apache : Accueil /var/www/html/index.html

Pour jouer, remplacez l'accueil ci-dessus comme suit :

\[srvdmz@srvdmz:~$\] cd /var/www/html 
\[srvdmz@srvdmz:~$\] sudo  mv index.html index.html.origine
\[srvdmz@srvdmz:~$\] sudo  touch index.html
\[srvdmz@srvdmz:~$\] sudo  nano index.html

et remplissez le nouvel index.html avec ce contenu :

<!DOCTYPE html>
<html>
<head><meta charset="utf-8"></head>
<body style="background-color:#FFEEE4">

<h1 style="color:#000000">
Accueil Web de l'entreprise Loup Virtuel
</h1>

<p style="font-size:30px;color:#CE6D39">
Toute une équipe est là pour répondre à vos besoins.
</p>

</body>
</html>

Pour finir, testez de nouveau l'URL :  
http://192.168.4.2

[![Capture - Apache : Page d'accueil personnalisée](/wp-content/uploads/2019/11/srvdmz_apache_nouvel_accueil-430x165.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/11/srvdmz_apache_nouvel_accueil.jpg)

Apache : Accueil personnalisé du serveur Web

#### _2.3 - PHP_

Installez le paquet php :

\[srvdmz@srvdmz:~$\] sudo apt install php

Créez ensuite un fichier de contrôle phpinfo.php :

\[srvdmz@srvdmz:~$\] sudo touch /var/www/html/phpinfo.php

Editez celui-ci :

\[srvdmz@srvdmz:~$\] sudo nano /var/www/html/phpinfo.php

et entrez le contenu ci-dessous :

<?php phpinfo(); ?> 

Puis, depuis la VM srvlan, testez l'URL :  
http://192.168.4.2/phpinfo.php

[![Capture - PHP : Page Web Infos PHP](/wp-content/uploads/2019/11/srvdmz_infos_php-430x262.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/11/srvdmz_infos_php.jpg)

PHP : Page Web Infos PHP

Les Cdes suivantes afficheront la version PHP ainsi que la liste des modules PHP installés :

\[srvdmz@srvdmz:~$\] php -v
\[srvdmz@srvdmz:~$\] php -m

#### _2.4 - MySQL_

MariaDB est un moteur de Bdd reposant sur MySQL.

Installez les 2 paquets mariadb suivants :

\[srvdmz@srvdmz:~$\] sudo apt install mariadb-server 
\[srvdmz@srvdmz:~$\] sudo apt install mariadb-client 

Vérifiez le démarrage automatique du serveur MySQL :

\[srvdmz@srvdmz:~$\] sudo systemctl status mariadb 

Retour :

```
● mariadb.service - MariaDB 10.3.18 database server
   Loaded: loaded (/lib/systemd/system/mariadb.se...
   Active: active (running) since Thu 2019-11... ago
    Docs: man:mysqld(8)
          https://mariadb.com/kb/en/library/systemd/
 Main PID: 1469 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 30 (limit: 545)
   Memory: 48.8M
   CGroup: /system.slice/mariadb.service
           └─1469 /usr/sbin/mysqld

... srvdmz systemd[1]: Starting MariaDB 10.3.18 ...
... srvdmz mysqld[1469]: ... /usr/sbin/mysqld...
... srvdmz /etc/mysql/debian-start[1503]: Upgrad...
... srvdmz systemd[1]: Started MariaDB ... server.
... srvdmz /etc/mysql/debian-start[1585]: Trigger...
```

puis modifiez en partie ses paramètres de sécurité :

\[srvdmz@srvdmz:~$\] sudo mysql\_secure\_installation 

en traitant le retour de la Cde ci-dessus comme suit :

```
NOTE: RUNNING ALL PARTS OF THIS ... FOR ALL MariaDB
      SERVERS IN PRODUCTION USE! PLEAS... CAREFULLY!

In order to log into MariaDB to secur... the current
password ...  If you've just installed MariaDB, and
you haven't set the root password yet, ... be blank,
so you should just press enter here.

Enter ... password for root ... : Appuyez sur Entrée
OK, successfully used password, moving on...

Setting the root password ... log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y      
New password: Votre MDP pour le root de MySQL
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installa..., allowing anyone
to log into MariaDB without ... account created for
them.  This is intended ... to make the installation
go a bit smoother.  You shou... before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success! Accès anonymous désactivé

Normally, root should ... from 'localhost'.  This
ensures that ... the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping. Accès root distant conservé

By default, MariaDB ... named 'test' that anyone can
access.  This is also ...ting, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] n 
 ... skipping. Bdd test conservée pour des tests

Reloading the privilege tables ... made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed ... your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Testez une connexion locale en tant que root MySQL :

\[srvdmz@srvdmz:~$\] sudo mysql -u root -p
\[sudo\] Mot de passe de srvdmz :
Enter password: _Peu importe, MariaDB gérant par défaut root avec son plugin unix\_socket_

Retour :

```
Welcome to the MariaDB monitor. Commands end with...
Your MariaDB connection id is 53
Server version: 10.3.18-MariaDB-0+deb10u1 Debian 10 

Copyright (c) 2000, 2018, Oracle, MariaDB Corpora...

Type 'help;' or '\h' for help. Type '\c' to clear ..

MariaDB [(none)]> \q
Bye
```

Un prompt d'accès à la console MysQL s'affiche.  
Utilisez la Cde \\q pour quitter.

Pour tester une connexion distante, commencez par éditez le fichier 50-server.cnf :

\[srvdmz@srvdmz:~$\] cd /etc/mysql/mariadb.conf.d
\[srvdmz@srvdmz:~$\] sudo nano 50-server.cnf

et remplacez la ligne bind-address = 127.0.0.1 par :

bind-address = 192.168.4.2          \# IP du serveur MySQL

Reconnectez-vous localement et créez une base de données de nom parc\_vm :

MariaDB \[(none)\]> CREATE DATABASE parc\_vm;

Touche Entrée après le point-virgule.

Retour :

```
Query OK, 1 row affected (0.00sec)

MariaDB [(none)]> 
```

Créez maintenant un utilisateur mysqluserdist autorisé à gérer celle-ci depuis la VM srvlan :

MariaDB \[(none)\]> GRANT ALL PRIVILEGES ON parc\_vm.\* TO mysqluserdist@'192.168.2.2' IDENTIFIED BY '_le MDP que vous voulez_ ';

Touche Entrée après le point-virgule.

Retour :

```
Query OK, 0 row affected (0.03sec)

MariaDB [(none)]> quit
```

Redémarrez MySQL :

\[srvdmz@srvdmz:~$\] sudo systemctl restart mariadb   

Allez sur srvlan, installez le paquet mariadb-client :

\[srvlan@srvlan:~$\] sudo apt install mariadb-client

et testez une connexion distante sur le serveur MySQL :

\[srvlan@srvlan:~$\] mysql -u mysqluserdist -p -h 192.168.4.2
Enter password: _Entrez le MDP de l'utilisateur mysqluserdist_

Retour :

```
Welcome to the MariaDB monitor. Commands end with...
Your MariaDB connection id is 36
Server version: 10.3.18-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corpora...

Type 'help;' or '\h' for help. Type '\c' to clear ..

MariaDB [(none)]>
```

Listez les Bdd visibles pour mysqluserdist :

![Image - MySQL : Retour de la Cde SHOW DATABASES](/wp-content/uploads/2022/03/liste_bdd_srvlan_debian10.webp)

### 3 - Installation du gestionnaire de Bdd Adminer

Cet outil plus léger que phpMyAdmin permettra d'administrer les Bdd du serveur MySQL facilement, ceci en local ou à distance via une interface Web.

Installez le paquet adminer :

\[srvdmz@srvdmz:~$\] sudo apt install adminer

Informez maintenant Apache de l'emplacement des fichiers du gestionnaire Adminer.

Pour cela, créez un fichier Apache adminer.conf :

\[srvdmz@srvdmz:~$\] cd /etc/apache2/conf-available 
\[srvdmz@srvdmz:~$\] sudo nano adminer.conf 

et entrez l'alias _(raccourci)_ suivant :

Alias /adminer /usr/share/adminer/adminer/

Demandez à Apache de prendre en compte le fichier :

\[srvdmz@srvdmz:~$\] sudo a2enconf adminer 

et rechargez la configuration pour traitement :

\[srvdmz@srvdmz:~$\] sudo systemctl reload apache2

#### _3.1 - Connexion distante sur les Bdd MySQL_

Vérifiez que MariaDB utilise bien par défaut le plugin unix\_socket pour root soit une authentification traitée en tant qu'utilisateur Unix et non par mot de passe.

Pour cela, connectez-vous :

\[srvdmz@srvdmz:~$\] sudo mysql -u root -p

et exécutez les instructions MySQL suivantes :

![Image - MySQL : Retour table mysql.user](/wp-content/uploads/2022/03/mysql_root_unix_socket.webp)

Cette configuration unix\_socket n'autorise que les connexions locales avec sudo et interdit les connexions distantes en tant que root.

Pour modifier ce comportement, reconnectez-vous :

\[srvdmz@srvdmz:~$\] sudo mysql -u root -p   

et exécutez les instructions MySQL suivantes :

```
[sudo] Mot de passe de srvdmz : 
Enter password:Peu importe, unix_socket étant actif 
Welcome ...

MariaDB [(none)]> Use mysql;
Reading table information for completion of tabl...
You can turn off this feature to get a quicker s...

Database changed
MariaDB [mysql]> UPDATE user SET plugin='' WHERE User='root';

Query OK, 1 row affected (0.006 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [mysql]> FLUSH PRIVILEGES;

Query OK, 0 rows affected (0.007 sec)

MariaDB [mysql]> quit
Bye
```

Pour terminer, depuis debian10-vm1, testez l'URL :  
http://192.168.4.2/adminer

[![Capture - Adminer : Fenêtre de connexion](/wp-content/uploads/2019/11/srvdmz_adminer_connexion_distante_1-430x203.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/11/srvdmz_adminer_connexion_distante_1.jpg)

Adminer : Fenêtre de connexion

Utilisez le MDP pour root créé au [§ 2.4](/site-web-php-mysql-debian10/#24_-_MySQL).

[![Capture - Adminer : Fenêtre d'administration](/wp-content/uploads/2019/11/srvdmz_adminer_connexion_distante_2-430x233.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/11/srvdmz_adminer_connexion_distante_2.jpg)

Adminer : Fenêtre d'administration

### 4 - Test global du serveur Web PHP MySQL

Ce test fera appel à un script PHP que vous allez écrire et exécuter.

Le script ouvrira une connexion sur la Bdd parc\_vm créée ci-dessus et permettra d'afficher sur une page Web le contenu d'une table de nom vm.

Connectez-vous localement sans utiliser la Cde sudo :

\[srvdmz@srvdmz:~$\] mysql -u root -p
Enter password : _MDP root créé au paragraphe 2.4_

et entrez les Cdes SQL suivantes pour créer, remplir et afficher la table vm :

MariaDB \[none\]> USE parc\_vm;

MariaDB \[parc\_vm\]> CREATE TABLE vm (Ligne INTEGER NOT NULL PRIMARY KEY);

MariaDB \[parc\_vm\]> ALTER TABLE vm ADD COLUMN Hôte VARCHAR(20);
MariaDB \[parc\_vm\]> ALTER TABLE vm ADD COLUMN OS VARCHAR(20);
MariaDB \[parc\_vm\]> ALTER TABLE vm ADD COLUMN Zone VARCHAR(10); 

MariaDB \[parc\_vm\]> INSERT INTO vm VALUES (1,'srvsec','Linux From Scratch','WAN');
MariaDB \[parc\_vm\]> INSERT INTO vm VALUES (2,'srvdmz','Debian 10','DMZ');
MariaDB \[parc\_vm\]> INSERT INTO vm VALUES (3,'srvlan','Debian 10','LAN');
MariaDB \[parc\_vm\]> INSERT INTO vm VALUES (4,'ovs','Debian 10','LAN');
MariaDB \[parc\_vm\]> INSERT INTO vm VALUES (5,'debian10-vm1','Debian 10','LAN');
MariaDB \[parc\_vm\]> INSERT INTO vm VALUES (6,'debian10-vm2','Debian 10','LAN');

MariaDB \[parc\_vm\]> SELECT \* FROM vm;

Retour de la dernière requête SQL :

![Image - MySQL : Retour de la Cde SELECT * FROM vm](/wp-content/uploads/2022/03/contenu_table_vm.webp)

Maintenant, créez et éditez le script testweb.php :

\[srvdmz@srvdmz:~$\] cd /var/www/html
\[srvdmz@srvdmz:~$\] sudo touch testweb.php
\[srvdmz@srvdmz:~$\] sudo nano testweb.php  

puis entrez le code HTML et PHP suivant :

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Test global du serveur Apache de la VM srvdmz</title>
</head>

<body style="background-color:lightyellow">
<h2 style="color:blue">Test global du serveur Apache</h2>
<p>Exemple d'une page Web dynamique :</p>
<p>Apache affiche les réponses de MySQL à propos<br />
de la Bdd 'parc\_vm' interrogée par PHP.</p><br />
<table border=1>

<caption>Affichage de la table 'vm'</caption>
<tr>
<th>Ligne :</th><th>Hôte :</th><th>OS :</th> <th>Zone :</th>
</tr>

<?php
// Connexion sur le serveur MySQL
$srv = "localhost"; $user = "root";
$pass = "votre MDP pour root"; $bdd = "parc\_vm";
$conn = mysqli\_connect($srv,$user,$pass,$bdd,) or die ("Connexion MySQL impossible...");

// Gestion correcte des accents inclus dans les requêtes
mysqli\_set\_charset($conn,"utf8");

// Sélection des enregistrements de la table 'vm' et
// envoi du résultat
$query = "SELECT \* FROM vm";
$result = mysqli\_query($conn,$query) or die ("Echec sur la table vm...");

// Insertion des lignes d'enregistrements trouvées
// dans un tableau HTML
while ($ligne = mysqli\_fetch\_array ($result))
{
echo "<tr><td>" . $ligne \["Ligne"\] . "</td>";
echo "<td>" . $ligne \["Hôte"\] . "</td>";
echo "<td>" . $ligne \["OS"\] . "</td>";
echo "<td>" . $ligne \["Zone"\] . "</td></tr>"; 
}

// Fermeture de la connexion
mysqli\_close($conn);
?>

</table>
</body>
</html>

Testez, cette fois depuis la VM srvlan, l'URL :  
http://192.168.4.2/testweb.php

[![Capture - Apache : Résultat de l'exécution du script testweb.php](/wp-content/uploads/2019/11/srvdmz_test_global_web-430x305.jpg "Cliquez pour agrandir l'image")](/wp-content/uploads/2019/11/srvdmz_test_global_web.jpg)

Apache : Résultat de l'exécution du script testweb.php

_Nota : Un mémento futur traitera de l'accès au serveur Web depuis Internet._

![Image - Rédacteur satisfait](/wp-content/uploads/2019/02/redacteur_satisfait_bis.png "Image Pixabay - Sanalinsan")

  
Voilà pour la création de base d'un  
site Web PHP MySQL. La partie 2  
vous attend pour la création d'un  
site WordPress sécurisé HTTPS.

[Partie 2](/https-et-wordpress/)
