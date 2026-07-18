---
title: "Windows - WSL2 et Debian"  
summary: Installation d'une VM Debian sous WSL2.  
authors:   
    - G.Leloup  
date: 2020-06-29  
categories:  
    - Windows
---

## WSL2 - Debian Xfce4

Debian est en service depuis 2020 _(Debian 10 sysvinit)_ avec une mise à jour continue jusqu'à la version Debian 13 incluant systemd, ceci sans problème.

### Préalables

Installer Debian depuis le Microsoft Store.

Lancer ensuite Debian depuis le menu de Windows.

Une console **bash** s'ouvre et l'installation de la VM commence :

-> Entrer un nom d'utilisateur comme demandé.

-> Entrer un MDP comme demandé.

-> Confirmer le MDP comme demandé.

L'installation continue et se termine.

### Locale fr_FR

Lancer la Cde suivante :

```bash
sudo dpkg-reconfigure locales  
```

Cocher le **fr_FR.UTF-8** et déclarer le français comme langue par défaut -> OK

La locale **fr_FR.UTF-8** est générée et peut être vérifiée comme suit :

```bash
locale -a
```

Déclarer la langue à utiliser :

```bash
sudo update-locale LANG=fr_FR.UTF8
```

Fermer et redémarrer ensuite la console **bash** pour prendre en compte la Cde ci-dessus.

Vérifier pour info la version de Debian installée :

```bash
cat /etc/debian_version
```

### Cdes utiles

A lancer depuis les terminaux **Windows PowerShell** ou **Terminal Windows**.

Etat de la VM Debian :

```bash
wsl --list --verbose
```

<!-- more -->

Retour :

```markdown
  NAME          STATE           VERSION
* kali-linux    Stopped         2
  Debian        Running         2
```

Démarrage de la VM Debian :

```bash
wsl -d Debian
```

Arrêt de la VM Debian :

```bash
wsl --shutdown Debian
```

Pour voir la liste des distributions Linux disponibles, entrer la Cde suivante :

```bash
wsl --list --online
```

Pour installer une distribution Linux, entrer la Cde suivante :

```bash
wsl --install -d Nom de la distribution Linux
```

Pour mettre à jour le noyau WSL :

```bash
wsl --update
```

Pour lire la version courante de WSL :

```bash
wsl --version
```

### Xfce4 - Installation

Lancer depuis le menu de **Windows Terminal** la **console** Debian.

Se rendre ensuite dans le dossier de l'utilisateur Debian créé lors de l'installation :

```bash
cd /home/utilisateur 
```

et lancer les Cdes suivantes :

```bash
 sudo apt update && sudo apt -y upgrade
 sudo apt -y install xfce4 xfce4-goodies
 sudo apt-get install xrdp
 sudo cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak
 sudo sed -i 's/3389/3390/g' /etc/xrdp/xrdp.ini
 sudo sed -i 's/max_bpp=32/#max_bpp=32\nmax_bpp=128/g' /etc/xrdp/xrdp.ini
 sudo sed -i 's/xserverbpp=24/#xserverbpp=24\nxserverbpp=128/g' /etc/xrdp/xrdp.ini
 echo xfce4-session > ~/.xsession
```  

Pour terminer, éditer le fichier de **xrdp** suivant :

```bash
sudo nano /etc/xrdp/startwm.sh
```  

Commenter ces 2 lignes _(# en début de ligne)_ :

```markdown
test -x /etc/X11/Xsession && exec /etc/X11/Xsession
exec /bin/sh /etc/X11/Xsession
```  

et ajouter celles-ci _(WSL étant en version 2.7.10.0 - 2026 )_ :

```markdown
# Force XFCE en X11 pour XRDP/WSL
unset WAYLAND_DISPLAY
export GDK_BACKEND=x11
export QT_QPA_PLATFORM=xcb
export XDG_SESSION_TYPE=x11

exec startxfce4
```

La ligne importante étant `unset WAYLAND_DISPLAY`, ceci pour éviter un conflit entre WSLg et XRDP qui utilise X11.

Démarrer ensuite le serveur **xrdp** :

```bash
sudo /etc/init.d/xrdp start  
```

et vérifier son statut :

```bash
sudo /etc/init.d/xrdp status 
```

Si OK :  
Relever l'IP de **eth0** avec la Cde **ip address**.

Ouvrir ensuite l'outil **Connexion Bureau à distance** de Windows.

Entrer dans le seul champ affiché :  
**IP-relevée:3390** ou **localhost:3390**.

Et cliquer sur le bouton **Connexion**.

Une fenêtre fournie par **xrdp** s'ouvre :  
Entrer le Login et MDP de l'utilisateur root de Debian.

Le bureau **xfce4** doit s'afficher :  
-> Utiliser les paramètres par défaut.

### Xfce4 - Locale fr_FR

Vérifier le statut courant avec la Cde suivante :

```bash
locale 
```

Retour à observer :

```markdown
LANG=fr_FR.UTF-8
LANGUAGE=
LC_CTYPE="fr_FR.UTF-8"
LC_NUMERIC="fr_FR.UTF-8"
LC_TIME="fr_FR.UTF-8"
LC_COLLATE="fr_FR.UTF-8"
LC_MONETARY="fr_FR.UTF-8"
LC_MESSAGES="fr_FR.UTF-8"
LC_PAPER="fr_FR.UTF-8"
LC_NAME="fr_FR.UTF-8"
LC_ADDRESS="fr_FR.UTF-8"
LC_TELEPHONE="fr_FR.UTF-8"
LC_MEASUREMENT="fr_FR.UTF-8"
LC_IDENTIFICATION="fr_FR.UTF-8"
LC_ALL= 
```

Pour info, lister les langues actuellement générées et disponibles sur Debian :

```bash
locale -a 
```

Retour possible :

```markdown
C
C.UTF-8
en_US.utf8
fr_FR.utf8
POSIX  
```

Si problème de langue rencontré, modifier la locale comme suit :

```bash
sudo dpkg-reconfigure locales (cocher le fr_FR.UTF-8)
sudo locale-gen (génère le français)
sudo update-locale LANG=fr_FR.UTF8
```  

En cas de problème persistant, essayer :

```bash
export LANG=fr_FR.UTF-8
export LANGUAGE=fr_FR.UTF-8
export LC_ALL=fr_FR.UTF-8
```  

Vérifier également le contenu du fichier **locale** :

```bash
cat /etc/default/locale 
```

Retour :

```markdown
#  File generated by update-locale
LC_ALL=fr_FR.UTF-8
LANGUAGE=fr_FR.UTF-8
LANG=fr_FR.UTF-8
```  

### xrdp - Démarrage auto

Ajouter à la fin du fichier **/home/utilisateur/.bashrc** le contenu suibant :

```markdown
# Démarrage de xrdp
sudo /etc/init.d/xrdp start
```

Ce contenu n'est plus utile depuis l'installation de systemd.

Modifier ensuite la section **[Logging]** du fichier **/etc/xrdp/xrdp.ini** comme suit :

Remplacer **DEBUG** par **INFO** dans les rubriques **LogLevel** et **SyslogLevel**.

### xrdp - Eviter le MDP

Entrer la Cde suivante :

```bash
sudo visudo
```

et ajouter à la fin du fichier la ligne :

```bash
%sudo ALL=(ALL) NOPASSWD: /etc/init.d/xrdp 
```

### Xfce4 - Synaptic

Installer le paquet suivant :

```bash
sudo apt install synaptic  
```

Ajouter ensuite un alias dans le fichier **/home/utilisateur/.bashrc** :

```markdown
# Démarrage de synaptic
alias synaptic="xhost +si:localuser:root ; sudo /usr/sbin/synaptic ; xhost -si:localuser:root"
```  

Pour lancer **synaptic** depuis la console Terminal :

```bash
synaptic  
```

Info sécurité :  
xhost **+** désactive le contrôle d'accès au serveur X.  
xhost **-** le réactive.

### Xfce4 - Firefox

Installer depuis **synaptic** les paquets **firefox-esr** et **firefox-esr-i10n-fr**.

### Debian - Démarrage

La VM démarre et présente malheureusement par défaut un dossier de Windows.

Pour ouvrir un dossier Linux, éditer le fichier Windows d'extension **.json** suivant :

```bash
C:\Users\Gerard\AppData\Local\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json
```

et ajouter un **startingDirectory** dans la section contenant **"name": "Debian"** :

```markdown
{
 "guid": "{58ad8b0c-3e.......................}",
 "hidden": false,
 "name": "Debian d\u00e9marrage",
 "source": "Windows.Terminal.Wsl",
 "startingDirectory": "//wsl$/Debian/home/utilisateur/"
 },
```

Le fichier **.json** est aussi accessible depuis le menu de **Terminal Windows** dans **Paramètres**.

### Debian - Arrêt et Sauvegarde

Le fichier settings.json peut être complété ainsi :

```markdown
{
"commandline": "wsl --shutdown Debian",
"guid": "{3fdcace8-379f-498c-9da2-969d9e321ebc}",
"hidden": false,
"icon": "ms-appx:///ProfileIcons/{9acb9455-ca41-5af7-950f-6bca1bc9722f}.png",
"name": "Debian arr\u00eat"
},
{
"closeOnExit": "never",
"commandline": "wsl --export Debian D:\\Images_WSL2\\DebianBackup.tar",
"guid": "{b5a43fde-52bb-430c-8707-e79eab3c50f0}",
"hidden": false,
"icon": "ms-appx:///ProfileIcons/{9acb9455-ca41-5af7-950f-6bca1bc9722f}.png",
"name": "Debian sauvegarde"
},
```

L'arrêt et la sauvegarde pourront ainsi être lancé depuis le menu du terminal Windows.

Les guid peuvent être créés avec la Cde suivante depuis le terminal Windows lancé en tant qu'administrateur :  

```text
[guid]::NewGuid()
```

### Explorer Windows - Fichiers Linux

Entrer dans le gestionnaire de fichier **Explorer** de Windows le chemin réseau suivant :

```markdown
\\wsl$\
```

### Wsl2 - Accès distant

- Windows

Ajouter le port proxy suivant :

```bash
netsh interface portproxy add v4tov4 listenport=xxxx listenaddress=0.0.0.0 connectport=80 connectaddress=172.29.187.193  
```

xxxx = N° de port proxy à créer et ouvrir dans le pare-feu de Windows.  
172.29.187.193 = Adresse IP temporaire de la VM Debian

Vérifier le proxy activé :

```bash
netsh interface portproxy show v4tov4
```

Pour supprimer le port proxy :

```bash
netsh interface portproxy delete v4tov4 listenport=xxxx listenaddress=0.0.0.0 protocol=tcp
```

Pour voir la configuration réseau IPv4 détaillée :

```bash
netsh interface ipv4 show config
```

- Debian

Installation du paquet **net-tools** pour pouvoir exploiter la Cde **netstat**.

- Windows

Modifier le parefeu comme suit :

-> Paramètres Windows  
-> Mise à jour et sécurité -> Sécurité Windows  
-> Pare-feu et protection réseau  
-> Paramètres avancés

La fenêtre pare-feu Windows Defender s'ouvre.

-> Règles du trafic entrant -> Nouvelle règle...  
-> Type de règle -> Port -> Suivant  
-> TCP -> Ports locaux spécifiques  
-> Entrer un n° de port xxxx -> suivant  
-> Autoriser la connexion -> Suivant  
-> Décocher public -> Suivant  
-> Donner un nom à la règle _(Accès dist WSL2)_  
-> Terminer

La nouvelle règle apparaît dans la liste des **Règles de trafic entrant**.

Redémarrer Windows et la VM Debian.

Entrer ensuite depuis un PC du réseau local cette URL :

`http://192.168.x.y:xxxx`

192.168.x.y = PC hôte des VM WSL2  
xxxx = n° du port proxy

### Wsl2 - Accès distant IP temporaire

Créer le script PowerShell **WSL_Network.ps1** suivant :

```bash
 If (-NOT ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator"))

{   
$arguments = "& '" + $myinvocation.mycommand.definition + "'"
Start-Process powershell -Verb runAs -ArgumentList $arguments
Break
}

$remoteport = bash.exe -c "ip addr | grep -Ee 'inet.*eth0'"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( $found ){
  $remoteport = $matches[0];
} else{
  echo "The Script Exited, the ip address of WSL 2 cannot be found";
  exit;
}

$ports=@(xxxx);

iex "netsh interface portproxy reset";
for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy add v4tov4 listenport=$port connectport=80 connectaddress=$remoteport";
}
iex "netsh interface portproxy show v4tov4";
```  

Placer ensuite le script dans le dossier de l'utilisateur Windows.

Ensuite créer une tâche depuis le planificateur de tâches de Windows :

-> Créer une tâche... ,  

Une fenêtre s'ouvre :

- Onglet Général

-> Nom, Entrer WSL2_IP_FORWARD  
-> Cocher Exécuter même si aucun utilisa...  
-> Cocher Ne pas stocker le mot de passe...

- Onglet déclencheurs

-> Nouveau  
-> Lancer la tâche : A l'ouverture de session  
-> Cocher Tout utilisateur > OK

- Onglet Actions

-> Nouveau > Action : Démarrer un programme  
-> Programme/script, entrer powershell.exe  
-> Ajouter des arguments -> Entrer

```bash
-ExecutionPolicy Bypass c:\Users\utilisateur\WSL_Network.ps1
```

-> OK

Au prochain démarrage de windows, le script sera exécuté sans afficher de fenêtre PowerShell.

Le serveur Web de la VM Debian sera accessible depuis n'importe quel PC du réseau local :

`http://192.168.x.y:xxxx`

192.168.x.y = PC hôte des VM WSL2  
xxxx = n° du port proxy

### Fichier wsl.conf

Selon la configuration, exemple de contenu du fichier **/etc/wsl.conf** :

```markdown
# Options de démarrage WSL2
[automount]
enabled = true
root = /mnt/
options = "metadata,umask=22,fmask=11"
mountFsTab = true

[boot]
systemd=true
command="service webmin start"

[user]
default=user-x

# Options DNS
[network]
generateHosts = true
generateResolvConf = true 
```

**Fin.**
