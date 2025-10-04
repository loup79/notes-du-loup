---
title: Debian - Glances
summary: Glances est un outil de supervision système accessible depuis le Web.
authors: 
  - G.Leloup
date: 2025-10-03
categories: 
  - Debian
---

## Supervision avec Glances

Glances peut afficher en temps réel le % d'utilisation des CPU et GPU, le % d'utilisation des mémoires système et GPU ainsi que les températures des différents périphériques, le tout sur une page accessible depuis un navigateur Web.

Avant de l'installer, vérifier la présence sur l'hôte Debian des paquets _psutils_ et _lm-sensors_, à défaut les ajouter.

Puis, mettre à jour le système Debian :

```bash
sudo apt update
sudo apt upgrade
```

### Environnement Python

Installer les paquets suivants :

```bash
sudo apt install python3-pip python3-dev
sudo apt install python3-full
```

Le paquet _python3-venv_ sera ajouté comme dépendance.

Créer ensuite un dossier pour l'environnement Python :

```bash
cd /home/user-x/
mkdir -p Documents/glances/python
```

<!-- more -->

et générer l'environnement dans celui-ci :

```bash
cd Documents/glances/
python3 -m venv python
```

### Installation de Glances

Installer Glances comme ceci :

```bash
cd python
source ./bin/activate
```

Le prompt du terminal change :

```bash
(python).../python$ python3 -m pip install --upgrade pip
(python).../python$ pip3 install --upgrade glances[web]
(python).../python$ pip3 install netifaces
```

Vérifier la version de Glances et activer son serveur Web :

```bash
(python).../python$ glances -V
(python).../python$ glances -w
```

Ouvrir si nécessaire le port 61208 sur Debian et tester l'URL `http://localhost:61208`.

### Service systemd

Créer un fichier de service _systemd_ afin de démarrer le serveur Web de Glances au boot du système Debian :

```bash
sudo nano /etc/systemd/system/glances.service
```

et entrer le contenu suivant :

```markdown
[Unit]
Description=Glances - An eye on your system
After=network.target

[Service]
Type=simple
User=user-x
WorkingDirectory=/home/user-x/Documents/glances/python/
ExecStart=/home/user-x/Documents/glances/python/bin/python -m glances -w
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Recharger la configuration _systemd_ et redémarrer _glances_ :

```bash
sudo systemctl daemon-reload
sudo systemctl restart glances
sudo systemctl status glances
sudo systemctl enable glances
```

### Fichier glances.conf

Un fichier de configuration _glances.conf_ se trouve dans :
_/home/user-x/Documents/glances/python/share/doc/glances/_

L'utilité de la configuration ci-dessous est à contrôler.

```bash
sudo mkdir /etc/glances
cd /home/user-x/Documents/glances/python/share/doc/glances/
sudo cp glances.conf /etc/glances/
```

Editer le fichier _glances.conf_ :

```bash
sudo nano /etc/glances/glances.conf
```

et modifier la ligne suivante :

```markdown hl_lines="2"
 # Available stats are: cpu,mem,load,swap
 list=cpu,mem,load
```

comme suit :

```markdown hl_lines="2"
 # Available stats are: cpu,mem,load,swap
 list=cpu,mem,load,swap
```

Redémarrer le service glances et observer le résulat dans la page Web de Glances.

```bash
sudo systemctl restart glances
```

### Tableau de bord

Quelques remarques concernant le tableau de bord affiché :

\- Composite, Sensor 1,2 = Températures du dique nvme-pci  
\- Tctl = Température du CPU Ryzen _(k10temp-pci-00c3)_  
\- edge = temperature du GPU
