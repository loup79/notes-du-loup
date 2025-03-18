---
title: Debian - Glances
summary: Glances est un outil de supervision système accessible depuis le Web.
authors: 
  - G.Leloup
date: 2025-03-18
categories: 
  - Debian
---

## Supervision système avec Glances

Glances peut afficher en temps réel le % d'utilisation des CPU et GPU, le % d'utilisation des mémoires système et GPU ainsi que les températures des différents périphériques, le tout sur une seule page accessible depuis un navigateur Web.

Avant de l'installer, vérifier la présence sur l'hôte Debian des paquets _psutils_ et _lm-sensors_, à défaut les ajouter.

Puis, procéder ainsi pour installer Glances :

```bash
# bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/glances.sh)"
```

Le même script peut-être lancé pour désinstaller Glances par la suite.

Une fois installé, ouvrir si nécessaire le port 61208 sur Debian et tester l'URL `http://ip-debian:61208`.

Tester la Cde suivante :

```bash
# glances -V
```

Celle-ci doit retourner le chemin du fichier _glances.log_.

Copier le fichier /usr/local/share/doc/glances/glances.conf dans /etc/glances :

```bash
# mkdir /etc/glances
# cp /usr/local/share/doc/glances/glances.conf /etc/glances/
```

Editer ensuite la configuration :

```bash
# nano /etc/glances/glances.conf
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
# systemctl restart glances
```

Quelques remarques concernant le tableau de bord affiché :

\- Composite, Sensor 1,2 et 8 = Températures du dique nvme-pci  
\- Tctl = Température du CPU Ryzen _(k10temp-pci-00c3)_  
\- edge = temperature du GPU
