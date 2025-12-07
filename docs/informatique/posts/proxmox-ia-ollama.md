---
title: IA - Ollama sur Proxmox
summary: Intelligence artificielle, gestion des modèles LLM avec Ollama et Open WebUI.
authors: 
  - G.Leloup
date: 2025-12-07
categories: 
  - Intelligence artificielle
---

## Intelligence Artificielle

L'outil **Ollama** démocratise l'accès aux modèles de langage de grande taille _(LLMs)_ en permettant aux utilisateurs de les exécuter localement.

Les modèles téléchargés peuvent être personnalisés et affinés afin de répondre à des objectifs précis, mais ceci exige une configuration matérielle solide.

La configuration matérielle présentée ci-dessous permettra d'exploiter des modèles de petites tailles _(SLMs)_ de l'ordre de 4B à 7B mais pas de personnaliser ceux-ci.

### Base de travail

Un mini PC équipé d'un CPU AMD Ryzen 7500U, 32 Go de RAM et une puce graphique intégrée _(iGPU)_ AMD Radeon Vega 8 offrant 3 Go de VRAM max.

L'**A**rchitecture de **M**émoire **U**nifiée est activée dans le BIOS, l'iGPU et le CPU partagent ainsi le même espace RAM ce qui augmente l'efficacité de traitement des grandes quantités de données exigée par l'utilisation de l'**I**ntelligence **A**rtificielle.

Système d'exploitation **Proxmox** 8.3.3 incluant un noyau linux 6.8.12.8-pve.

Ci-dessous, quelques Cdes utiles pour appréhender la suite :

-\ Cdes pour obtenir les informations de l'iGPU :

```bash
proxmox:~# apt install rocminfo
proxmox:~# rocminfo
```

Retour :

```markdown hl_lines="4"
...
Agent 2
*******
  Name:                    gfx90c
  Uuid:                    GPU-XX
  Marketing Name:          AMD Radeon Graphics
  Vendor Name:             AMD
...
```

L'étiquette ==gfx90c== désigne l'iGPU AMD Radeon Vega 8.

<!-- more -->

\- Cdes pour observer l'activité en temps réel d'un iGPU AMD :

```bash
proxmox:~# apt install radeontop
proxmox:~# radeontop
```

Touches CTRL+C pour quitter.

\- Cdes pour obtenir des informations sur la mémoire vidéo :

```bash
proxmox:~# cat /sys/class/drm/card1/device/mem_info_vram_total
proxmox:~# cat /sys/class/drm/card1/device/mem_info_vram_used
```

Retours :

```markdown
3221225472  # 3 Go de VRAM alloué
154243072   # 150 Mo de VRAM utilisé
```

\- Cde pour obtenir le nom du pilote graphique exploité :

```bash
proxmox:~# lspci -k | grep -A 3 "VGA"
```

Retour :

```markdown hl_lines="3"
04:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Lucienne (rev c1)
  Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Lucienne
  Kernel driver in use: amdgpu
  Kernel modules: amdgpu
```

Les firmwares des GPU de marque AMD sont stockés dans le dossier _/lib/firmware/amdgpu/_.

\- Cdes pour obtenir les ID des groupes _video_ et _render_ :

```bash
proxmox:~# cat /etc/group | grep video
proxmox:~# cat /etc/group | grep render
```

Retours :

```markdown
44
104
```

\- Cde pour obtenir la version courante de Python :

```bash
proxmox:~# python3 --version
```

Retour :

```markdown
Python 3.11.2
```

### Conteneur LXC ubuntu-ollama

#### _- Création (script ubuntu.sh)_

Accédez à l'interface Web de Proxmox et créez le conteneur en exécutant le script [tteck](https://github.com/tteck/Proxmox/tree/main/ct/ubuntu.sh){ target="_blank" } suivant depuis la console du noeud Proxmox :

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"
```

Un menu dédié au script s'ouvre, accédez au mode avancé puis _Sélectionnez_ ou _Entrez_ :  
\- L'OS Ubuntu 22.04 Jammy  
\- Un conteneur LXC privilégié  
\- Le MDP de l'utilisateur _root_  
\- L'ID du conteneur _Ex: 112_  
\- Le nom d'hôte _Ex: ubuntu-ollama_  
\- La taille du disque dur _Ex: 120 Go_  
\- Le nombre de coeurs alloué au conteneur _Ex: 4_  
\- La taille de la RAM _Ex: 8000 Mo_  
\- Le bridge réseau vmbr0  
\- Une adresse IP statique ou dynamique  
\- L'IP de la gateway  _Ex: 192.168.x.z_  
\- La désactivation de l'IPV6  
\- L'IP de votre serveur DNS local _Ex: 192.168.x.z_  
\- La validation d'un accès SSH en tant que _root_  
\- Le mode verbeux pour suivre la création du conteneur

Si tout se passe bien :

```markdown
...
Lecture des informations d'état... Fait      
 ✓ Cleaned
 ✓ Completed Successfully!
```

Lancez ensuite une connexion SSH sur le conteneur et mettez à jour le système Ubuntu :

```bash
root@ubuntu-ollama:~# apt update
root@ubuntu-ollama:~# apt upgrade
```

Vérifiez la version du noyau linux de l'hôte Proxmox :

```bash
root@ubuntu-ollama:~# uname -r
```

Retour :

```markdown
6.8.12-8-pve
```

Contrôlez la version courante de Python :

```bash
root@ubuntu-ollama:~# python3 --version
```

Retour possible :

```markdown
Python 3.12.9
```

Vérifiez pour finir les ID des groupes _video_ et _render_ :

```bash
root@ubuntu-ollama:~# cat /etc/group | grep -w 'render\|\video'
```

retour :

```markdown
video:x:44
render:x:108
```

#### _- Création (script ollama.sh)_

Ce script plus récent que ci-dessus installe une base sous l'OS Ubuntu 24.04 :

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/ollama.sh)"
```

Pour les entrées du script à gérer, voir la section ci-dessus.

Changements par rapport aux sections ci-dessous :

-- Section _Ollama et Python_ --  
Environnement Python créé dans la même version que celle installée sur l'hôte Proxmox.

-- Section _ROCm™_ --  
ROCm non installé car non fonctionnel avec un conteneur LXC et iGPU.

Editez la configuration du conteneur créé :

```bash
proxmox:~# nano /etc/pve/lxc/id-du-conteneur.conf
```

et modifiez son contenu comme suit, n'ajoutez rien d'autre :

```markdown
#<div align='center'>
Gardez le contenu intérieur de la balise DIV
#</div>
arch: amd64
cores: 4
features: nesting=1
hostname: ubuntu-ollama
memory: 8192
nameserver: 192.168.x.z
net0: name=eth0,bridge=vmbr0,hwaddr=BC:27:BA:C9:38:21,ip=dhcp,type=veth
onboot: 1
ostype: ubuntu
rootfs: local-lvm:vm-<id-du-conteneur>-disk-0,size=120G
swap: 2048
tags: ai;community-script
lxc.cgroup2.cpuset.cpus: 0-15 
lxc.apparmor.profile: unconfined
```

Ajustez les valeurs de _cores_, _memory_, _nameserver_, _hwaddr_, _size_ et _lxc.cgroup2.cpuset.cpus_ selon votre configuration.

Fixez la taille du swap à 2Go minimum.

La ligne _lxc.cgroup2.cpuset.cpus: 0-15_ est importante. Elle implique que les 4 coeurs sur les 16 du CPU Ryzen 7 qui seront attribués aléatoirement aux conteneur LXC seront pleinement exploités par Ollama.

-- Section _Installation d'Ollama_ --  
Ollama est installé automatiquement depuis le script.

Vérifiez le ainsi :

```bash
(base) root@ubuntu-ollama:~# ollama -v
```

Ensuite modifiez le fichier /etc/systemd/system/ollama.service comme suit :

```markdown
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
Type=exec
ExecStart=/usr/local/bin/ollama serve
# CPUQuota à 280% = 70% max de 4 VCPU
CPUQuota=280%
Environment=HOME=/root
Environment=OLLAMA_CONFIG_DIR=/etc/ollama
Environment="PATH=/root/miniconda3/envs/ollamaenv/bin:/root/miniconda3/condabin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:"
Environment="OLLAMA_HOST=0.0.0.0"
# Forcer l'utilisation maximale des threads CPU
Environment=OMP_NUM_THREADS=4
Environment=MKL_NUM_THREADS=4
Environment=OPENBLAS_NUM_THREADS=4
Environment=NUMEXPR_NUM_THREADS=4
# Délai d'inactivité avant arrêt du modèle
Environment=OLLAMA_KEEP_ALIVE=5m
# Relancer automatiquement si plantage
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

-- Section _Mise à jour d'Ollama_ --  
Aucun changement.

-- Section _Installation d'Open WebUI_ --  
Pas de changement pour toute la partie Open WebUI sauf l'utilisation de python 3.12.12 à la place de python 3.11.2.

-- Section _Ajout de modèles d'IA_ --  
Aucun changement.

-- Section _Mise à jour d'Oen WebUI_ --  
Aucun changement.

Au final, que ce soit le script de _tteck_ ou celui-ci, les modèles d'IA 4B q4_K_M tourneront correctement sur une quantité de RAM 8 Go allouée à Ollama.

Des modèles 3B q5_K_M donneront également de bons résultats, q5_K_M offrant une quantification meilleure que q4_K_M.

#### _- Ollama et Python_

Ollama offre une suite d’outils compatibles avec Python et une API étendue, ce qui permet de créer, gérer et déployer des modèles d’IA.

Site de référence pour la suite : [Anaconda](https://www.anaconda.com/docs/getting-started/miniconda/install#macos-linux-installation){ target="_blank" }

Installez Miniconda, outil qui permettra de créer l'environnement virtuel Python dédié à Ollama :

```bash
root@ubuntu-ollama:~# cd /root
~# mkdir miniconda3
~# cd miniconda3

~# wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

~# mv Miniconda3-latest-Linux-x86_64.sh miniconda.sh
~# bash miniconda.sh -b -u -p /root/miniconda3
~# rm miniconda.sh
```

Editez ensuite le fichier _.bashrc_ de l'utilisateur _root_ :

```bash
root@ubuntu-ollama:~# cd /root
root@ubuntu-ollama:~# nano .bashrc
```

et ajoutez en fin de celui-ci le chemin vers la Cde conda :

```markdown
# Ajout du chemin d'accès au programme conda
export PATH=/root/miniconda3/bin:$PATH
```

Fermez la connexion SSH courante afin de tester la modification, ouvrez en une autre, puis :

```bash
root@ubuntu-ollama:~# conda --version
```

Retour :

```markdown
conda 25.1.1
```

Enfin, créez l'environnement dans une version Python compatible avec Ollama telle la 3.11.2 :

```bash
root@ubuntu-ollama:~# cd /root/miniconda3/bin
root@ubuntu-ollama:~# conda create --prefix /root/miniconda3/envs/ollamaenv python=3.11.2
root@ubuntu-ollama:~# conda init --all
```

La dernière Cde modifie automatiquement le fichier _.bashrc_ et il est possible de retirer les 2 lignes ajoutées précédemment.

Fermez de nouveau la connexion SSH courante, ouvrez en une autre, puis activez le nouvel environnement :

```bash
root@ubuntu-ollama:~# conda activate ollamaenv
(ollamaenv) root@ubuntu-ollama:~#
```

Le prompt change, gardez l'environnement Python ouvert.

!!! note "Nota"
    Utilisez la Cde "conda deactivate" pour quitter un environnement python.

#### _- ROCm™_

L'installation de ROCm est montrée à titre indicatif car dans le cas d'un conteneur LXC créé sur un hôte avec **iGPU** celle-ci s'avère inutile, l'iGPU ne sera pas pleinement exploité.

Ollama peut fonctionner sans problème sur CPU seule.

**1)** ROCm™ _(Radeon Open Compute)_ est nécessaire pour tirer parti des capacités de calcul d'un GPU AMD _(pas d'un iGPU)_ et doit être installé dans le conteneur.

Site de référence pour la suite : [ROCm](https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native_linux/install-radeon.html){ target="_blank" }

Installez celui-ci à l'aide du script _amdgpu-install_ :

```bash
(ollamaenv) root@ubuntu-ollama:~# cd /root
(ollamaenv) root@ubuntu-collama:~# wget https://repo.radeon.com/amdgpu-install/6.3.2/ubuntu/jammy/amdgpu-install_6.3.60302-1_all.deb

(ollamaenv) root@ubuntu-ollama:~# dpkg -i amdgpu-install_6.3.60302-1_all.deb
(ollamaenv) root@ubuntu-ollama:~# amdgpu-install --no-dkms --usecase=graphics,rocm,opencl -y
(ollamaenv) root@ubuntu-ollama:~# rm amdgpu-install_6.3.60302-1_all.deb
```

L'installation de ROCm 6.3.2 doit se dérouler sans échec.

Le flag _--no-dkms_ évite d'installer les pilotes du noyau linux, car ceux-ci sont déjà gérés par l'hôte Proxmox.

**2)** Le bon fonctionnement de ROCm nécessite de définir une variable d'environnement.

Pour cela, éditez le fichier _.bashrc_ :

```bash
(ollamaenv) root@ubuntu-ollama:~# nano /root/.bashrc
```

et ajoutez ce contenu en fin de fichier :

```markdown
# Gestion du GPU AMD Radeon gfx90c
export HSA_OVERRIDE_GFX_VERSION=9.0.0
```

**3)** Ci-dessous, pour info, un exemple de configuration de conteneur LXC inluant le montage des périphériques nécessaires à ROCm.

```bash
proxmox:~# cat /etc/pve/lxc/id-du-conteneur.conf
```

```markdown
arch: amd64
cores: 4
memory: 8192
swap: 2048
rootfs: local-lvm:vm-<ID_CT>-disk-0,size=32G
ostype: ubuntu
hostname: ubuntu-ollama
unprivileged: 0

# Réseau
net0: name=eth0,bridge=vmbr0,hwaddr=BC:27:BA:C9:38:21,ip=dhcp,type=veth

# Montage des périphériques pour ROCm
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.mount.entry: /dev/kfd dev/kfd none bind,optional,create=file
lxc.cgroup2.devices.allow: c 226:* rwm
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
lxc.mount.entry: /dev/dri/card0 dev/dri/card0 none bind,optional,create=file
```

Pour finir, redémarrez le conteneur `ubuntu-ollama`.

**4)** Contrôles

Ouvrez une connexion SSH et testez les Cdes suivantes :

```bash
(base) root@ubuntu-ollama:~# conda activate ollamaenv
(ollamaenv) root@ubuntu-ollama:~# python3 --version
(ollamaenv) root@ubuntu-ollama:~# rocminfo
(ollamaenv) root@ubuntu-ollama:~# rocm-smi
(ollamaenv) root@ubuntu-ollama:~# clinfo
```

Installez le paquet radeontop et testez son fonctionnement :

```bash
(ollamaenv) root@ubuntu-ollama:~# apt install radeontop
(ollamaenv) root@ubuntu-ollama:~# radeontop
```

Touches CTRL+C pour quitter radeontop.

#### _- Installation d'Ollama_

Site de référence pour la suite : [Ollama](https://github.com/ollama/ollama/blob/main/docs/linux.md){ target="_blank" }

Installation manuelle :

```bash
(ollamaenv) root@ubuntu-ollama:~# cd /root
(ollamaenv) root@ubuntu-ollama:~# curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
(ollamaenv) root@ubuntu-ollama:~# tar -C /usr -xzf ollama-linux-amd64.tgz
```

Démarrez le serveur Ollama :

```bash
(ollamaenv) root@ubuntu-ollama:~# ollama serve
```

Ouvrez une seconde connexion SSH et vérifiez la version :

```bash
(base) root@ubuntu-ollama:~# ollama -v
```

retour :

```markdown
ollama version is 0.5.12
```

Créez un utilisateur et un groupe pour Ollama :

```bash
(base) root@ubuntu-ollama:~# useradd -r -s /bin/false -U -m -d /usr/share/ollama ollama
(base) root@ubuntu-ollama:~# usermod -a -G ollama $(whoami)
```

L'utilisateur _root_ a été ajouté au groupe _ollama_.

Ajoutez enfin l'utilisateur _ollama_ aux groupes _video_ et _render_ :

```bash
(base) root@ubuntu-ollama:~# usermod -a -G video ollama
(base) root@ubuntu-ollama:~# usermod -a -G render ollama
```

Créez un fichier de service Systemd pour Ollama :

```bash
(base) root@ubuntu-ollama:~# nano /etc/systemd/system/ollama.service
```

et entrez le contenu suivant :

```markdown
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/bin/ollama serve
# CPUQuota à 280% = 70% max de 4 VCPU
CPUQuota=280%
Environment=HOME=/root
Environment=OLLAMA_CONFIG_DIR=/etc/ollama
Environment="PATH=/root/miniconda3/envs/ollamaenv/bin:/root/miniconda3/condabin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:"
Environment="OLLAMA_HOST=0.0.0.0"
# Forcer l'utilisation maximale des threads CPU
Environment=OMP_NUM_THREADS=4
Environment=MKL_NUM_THREADS=4
Environment=OPENBLAS_NUM_THREADS=4
Environment=NUMEXPR_NUM_THREADS=4
# Délai d'inactivité avant arrêt du modèle
Environment=OLLAMA_KEEP_ALIVE=5m
# Seulement si utilisation GPU
#Environment="HSA_OVERRIDE_GFX_VERSION=9.0.0"
#Environment="ROCM_PATH=/opt/rocm"
# Relancer automatiquement si plantage GPU
Restart=always
RestartSec=3

[Install]
WantedBy=default.target
```

Validez le démarrage automatique du service :

```bash
(base) root@ubuntu-ollama:~# systemctl daemon-reload
(base) root@ubuntu-ollama:~# systemctl enable ollama
```

Puis, redémarrez le conteneur et vérifiez le statut d'Ollama :

```bash
(base) root@ubuntu-ollama:~# systemctl status ollama
```

Cdes Ollama utiles :

- ollama serve _(démarrage du serveur Ollama)_
- ollama --version  
- ollama pull `nom-du-modele` _(téléchargement)_
- ollama list _(liste des modèles installés)_
- ollama show `nom-du-modele` _(détail)_
- ollama run `nom-du-modele` _(démarrage)_  
- ollama stop `nom-du-modele` _(arrêt)_
- ollama rm `nom-du-modele` _(suppression)_

#### _- Mise à jour d'Ollama_

Procédez ainsi :

```bash
(ollamaenv) root@ubuntu-ollama:~# cd /root
(ollamaenv) root@ubuntu-ollama:~# rm -rf /usr/lib/ollama
(ollamaenv) root@ubuntu-ollama:~# curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
(ollamaenv) root@ubuntu-ollama:~# tar -C /usr -xzf ollama-linux-amd64.tgz
(ollamaenv) root@ubuntu-ollama:~# systemctl daemon-reload
(ollamaenv) root@ubuntu-ollama:~# systemctl restart ollama
(ollamaenv) root@ubuntu-ollama:~# ollama -v
```

#### _- Installation d'Open WebUI_

Open WebUI est une interface graphique pour Ollama.

Site de référence : [Open WebUI](https://pypi.org/project/open-webui/){ target="_blank" }

Préparez l'installation d'Open WebUI comme suit :

```bash
(base) root@ubuntu-ollama:~# cd /root
(base) root@ubuntu-ollama:~# conda create -n openwebuienv python=3.11.2
(base) root@ubuntu-ollama:~# conda activate openwebuienv
(openwebuienv) root@ubuntu-ollama:~#
```

Installez Open WebUI :

```bash
(openwebuienv) root@ubuntu-ollama:~# pip install open-webui
```

L'installation doit se dérouler sans échec.

Démarrez ensuite le serveur d'Open WebUI :

```bash
(openwebuienv) root@ubuntu-ollama:~# open-webui serve
```

Le premier démarrage peut être assez long car il semble impliquer par défaut le téléchargement d'un certain nombre de modèles, Ex: _model.safetensorrs_, _model_03.onnx_, etc...

Une fois terminé, ouvrez depuis un navigateur Web local l'URL `http://ip-ubuntu-ollama:8080` et créez le compte administrateur comme demandé.

Redémarrez le conteneur depuis Proxmox et créez le fichier suivant qui sera utilisé par la suite pour lancer automatiquement le serveur d'Open WebUI :

```bash
(base) root@ubuntu-ollama:~# nano /root/start_open-webui.sh
```

Entrez le contenu suivant :

```markdown
#!/bin/bash

# Activer l'environnement Conda de nom openwebuienv
source /root/miniconda3/etc/profile.d/conda.sh
conda activate /root/miniconda3/envs/openwebuienv

# Démarrage du serveur Open WebUI
open-webui serve
```

Puis, rendez le fichier exécutable :

```bash
(base) root@ubuntu-ollama:~# chmod +x start_open-webui.sh
```

Créez un fichier de service Systemd pour Open WebUI :

```bash
(base) root@ubuntu-ollama:~# nano /etc/systemd/system/openwebui.service
```

et entrez le contenu suivant :

```markdown
[Unit]
Description=Open WebUI Service
After=network.target

[Service]
Type=simple
ExecStart=/root/start_open-webui.sh
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Démarrez le service :

```bash
(base) root@ubuntu-ollama:~# systemctl daemon-reload
(base) root@ubuntu-ollama:~# systemctl start openwebui
(base) root@ubuntu-ollama:~# systemctl status openwebui
```

Si le statut est bon, validez le démarrage automatique :

```bash
(base) root@ubuntu-ollama:~# systemctl enable openwebui
```

Pour finir, redémarrez le conteneur et testez de nouveau l'URL `http://ip-ubuntu-ollama:8080`.

#### _- Ajout de modèles d'IA_

!!! Note "Nota"
    Le site [ollama.com](https://ollama.com/search){ target="_blank" } peut être utilisé pour découvrir les Cdes de téléchargement des modèles.

Téléchargez par exemple le modèle Mistral 7B comme suit :

```bash
(base) root@ubuntu-ollama:~# ollama pull mistral
(base) root@ubuntu-ollama:~# ollama list
```

Les modèles sont installés dans le dossier _/root/.ollama/models/_.

Ouvrez ensuite la pabe Web d'Open WebUI et jouez en testant le modèle de Mistral.

D'autres modèles peuvent être installés et testés, tels :

- llama3 8b, quantization Q4_0, _Meta_  
- llava-llama3 8b, quantization Q4_K_M, _XTuner_
- gemma2 2b, quantization Q4_0, _Google_  
- phi3 4b, quantization Q4_0, _Microsoft_

#### _- Mise à jour d'Open WebUI_

Procédez ainsi :

```bash
(base) root@ubuntu-ollama:~# cd /root/miniconda3/envs/openwebuienv
(base) root@ubuntu-ollama:~# conda activate openwebuienv
(openwebuienv) root@ubuntu-ollama:~# systemctl stop openwebui
(openwebuienv) root@ubuntu-ollama:~# pip install --upgrade open-webui
(openwebuienv) root@ubuntu-ollama:~# systemctl start openwebui
```

Vérification :

```bash
(openwebuienv) root@ubuntu-ollama:~# pip freeze | grep open-webui
```

Retour :

```markdown
open-webui==0.5.x
```

Une fois terminé, pensez à vider les caches de votre navigateur Web _(Cookies et données de sites)_ avant de vous connecter sur la page d'Open WebUI.

**Fin.**
