---
title: IA - ComfyUI sur Proxmox
summary: Intelligence artificielle, génération d'images avec ComfyUI.
authors: 
  - G.Leloup
date: 2025-02-27
categories: 
  - Intelligence artificielle
---

## Intelligence Artificielle

ComfyUI offre une interface utilisateur servant à générer des images à partir d'un texte.

Les utilisateurs peuvent créer des flux de travail complexes en utilisant différents nœuds personnalisés et modèles d'IA.

### Base de travail

Un CPU AMD Ryzen 7500U, 32 Go de RAM et une puce graphique intégrée _(iGPU)_ AMD Radeon Vega 8.

L'**A**rchitecture de **M**émoire **U**nifiée est activée dans le BIOS, l'iGPU et le CPU partagent ainsi le même espace RAM ce qui augmente l'efficacité de traitement des grandes quantités de données exigée par l'utilisation de l'**I**ntelligence **A**rtificielle.

Système d'exploitation **Proxmox** 8.3.3 incluant un noyau linux 6.8.12.8-pve.

Ci-dessous, quelques Cdes utiles pour appréhender la suite :

\- Cdes pour obtenir les informations d'un iGPU AMD :

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

### Conteneur LXC ubuntu-comfyui

!!! Warning "Attention aux tailles des disques"
    Prévoir une taille de disque dur supérieure de 20 Go par rapport à celle du conteneur Ollama soit 140 Go.

#### _- Création_

Accédez à l'interface Web de Proxmox et créez le conteneur en exécutant le script [tteck](https://github.com/tteck/Proxmox/tree/main/ct/ubuntu.sh){ target="_blank" } suivant depuis la console du noeud Proxmox :

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/ubuntu.sh)"
```

Un menu dédié au script s'ouvre, accédez au mode avancé puis _Sélectionnez_ ou _Entrez_ :  
\- L'OS Ubuntu 22.04 Jammy  
\- Un conteneur LXC privilégié  
\- Le MDP de l'utilisateur _root_  
\- L'ID du conteneur _Ex: 113_  
\- Le nom d'hôte _Ex: ubuntu-comfyui_  
\- La taille du disque dur _Ex: 140 Go_  
\- Le nombre de coeurs alloué au conteneur _Ex: 6_  
\- La taille de la RAM _Ex: 10000 Mo_  
\- Le bridge réseau vmbr0  
\- Une adresse IP statique de la forme 192.168.x.y/24  
\- L'IP de la gateway _Ex: 192.168.x.z_  
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
root@ubuntu-comfyui:~# apt update
root@ubuntu-comfyui:~# apt upgrade
```

Vérifiez la version du noyau linux de l'hôte Proxmox :

```bash
root@ubuntu-comfyui:~# uname -r
```

Retour :

```markdown
6.8.12-8-pve
```

Contrôlez la version courante de Python :

```bash
root@ubuntu-comfyui:~# python3 --version
```

Retour possible :

```markdown
Python 3.12.8
```

Vérifiez pour finir les ID des groupes _video_ et _render_ :

```bash
root@ubuntu-comfyui:~# cat /etc/group | grep -w 'render\|\video'
```

retour :

```markdown
video:x:44
render:x:108
```

#### _- ComfyUI et Python_

Python est essentiel pour ComfyUI en raison de sa capacité à automatiser, étendre et simplifier l'exécution des flux de travail.

Site de référence : [Anaconda](https://www.anaconda.com/docs/getting-started/miniconda/install#macos-linux-installation){ target="_blank" }

Installez Miniconda, outil qui permettra de créer l'environnement virtuel Python dédié à ComfyUI :

```bash
root@ubuntu-comfyui:~# cd /root
~# mkdir miniconda3
~# cd miniconda3

~# wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

~# mv Miniconda3-latest-Linux-x86_64.sh miniconda.sh
~# bash miniconda.sh -b -u -p /root/miniconda3
~# rm miniconda.sh
```

Editez ensuite le fichier _.bashrc_ de l'utilisateur _root_ :

```bash
root@ubuntu-comfyui:~# cd /root
root@ubuntu-comfyui:~# nano .bashrc
```

et ajoutez en fin de celui-ci le chemin vers la Cde conda :

```markdown
# Ajout du chemin d'accès au programme conda
export PATH=/root/miniconda3/bin:$PATH
```

Fermez la connexion SSH courante afin de tester la modification, ouvrez en une autre, puis :

```bash
root@ubuntu-comfyui:~# conda --version
```

Retour :

```markdown
conda 25.1.1
```

Enfin, créez l'environnement dans une version Python compatible avec ComfyUI telle la 3.12 :

```bash
root@ubuntu-comfyui:~# cd /root/miniconda3/bin
root@ubuntu-comfyui:~# conda create --prefix /root/miniconda3/envs/comfyenv python=3.12
root@ubuntu-comfyui:~# conda init --all
```

La dernière Cde modifie automatiquement le fichier _.bashrc_ et il est possible de retirer les 2 lignes ajoutées précédemment.

Fermez de nouveau la connexion SSH courante, ouvrez en une autre, puis activez le nouvel environnement :

```bash
root@ubuntu-comfyui:~# cd /root
root@ubuntu-comfyui:~# conda activate comfyenv
(comfyenv) root@ubuntu-comfyui:~#
```

Le prompt change, gardez l'environnement Python ouvert.

#### _- ROCm™_

**1)** ROCm™ _(Radeon Open Compute)_ est nécessaire pour tirer parti des capacités de calcul du GPU AMD et doit être installé dans le conteneur.

Site de référence : [ROCm](https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native_linux/install-radeon.html){ target="_blank" }

Installez celui-ci à l'aide du script _amdgpu-install_ :

```bash
(comfyenv) root@ubuntu-comfyui:~# cd /root
(comfyenv) root@ubuntu-comfyui:~# wget https://repo.radeon.com/amdgpu-install/6.3.2/ubuntu/jammy/amdgpu-install_6.3.60302-1_all.deb

(comfyenv) root@ubuntu-comfyui:~# dpkg -i amdgpu-install_6.3.60302-1_all.deb
(comfyenv) root@ubuntu-comfyui:~# amdgpu-install --no-dkms --usecase=graphics,rocm,opencl -y
(comfyenv) root@ubuntu-comfyui:~# rm amdgpu-install_6.3.60302-1_all.deb
```

L'installation de ROCm 6.3.2 doit se dérouler sans échec.

Le flag _--no-dkms_ évite d'installer les pilotes du noyau linux, car ceux-ci sont déjà installés sur l'hôte Proxmox.

**2)** Le bon fonctionnement de ROCm nécessite de définir une variable d'environnement.

Pour cela, éditez le fichier _.bashrc_ :

```bash
(comfyenv) root@ubuntu-comfyui:~# nano /root/.bashrc
```

et ajoutez ce contenu en fin de fichier :

```markdown
# Gestion du GPU gfx90c
export HSA_OVERRIDE_GFX_VERSION=9.0.0
```

**3)** L'accès direct aux ressources du GPU physique de l'hôte Proxmox nécessite de gérer le GPU _Passthrough_.

Depuis Proxmox, éditez la configuration du conteneur :

```bash
proxmox:~# nano /etc/pve/lxc/id-du-conteneur.conf
```

et ajoutez sous la ligne _cores: 10_ le contenu suivant :

```markdown
...
cores: 10
dev0: /dev/kfd,gid=44,uid=0
dev1: /dev/dri/renderD128,gid=108,uid=0
...
```

Sur l'interface graphique de Proxmox, les nouveaux paramètres apparaissent dans les _Ressources_ du conteneur.

Pour finir, redémarrez le conteneur `ubuntu-comfyui`.

**4)** Contrôles

Ouvrez une connexion SSH et testez les Cdes suivantes :

```bash
(base) root@ubuntu-comfyui:~# conda activate comfyenv
(comfyenv) root@ubuntu-comfyui:~# python3 --version
(comfyenv) root@ubuntu-comfyui:~# rocminfo
(comfyenv) root@ubuntu-comfyui:~# rocm-smi
(comfyenv) root@ubuntu-comfyui:~# clinfo
```

Installez le paquet radeontop et testez son fonctionnement :

```bash
(comfyenv) root@ubuntu-comfyui:~# apt install radeontop
(comfyenv) root@ubuntu-comfyui:~# radeontop
```

Touches CTRL+C pour quitter.

#### _- PyTorch_

PyTorch permet entres autres d'exécuter des calculs intensifs sur les GPU, ce qui est crucial pour des tâches comme la génération d'images à partir de modèles de diffusion.

Sans PyTorch, ces calculs seraient bien plus lents, car ils se feraient uniquement sur le CPU.

Site de référence : [PyTorch](https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native_linux/install-pytorch.html){ target="_blank" }

Le bon fonctionnement de Pytorch nécessite de définir une variable d'environnement.

Pour cela, éditez le fichier _.bashrc_ :

```bash
(comfyenv) root@ubuntu-comfyui:~# nano /root/.bashrc
```

et ajoutez ceci sous _export ..._GFX_VERSION=9.0.0_ :

```markdown hl_lines="3"
# Gestion du GPU gfx90c
export HSA_OVERRIDE_GFX_VERSION=9.0.0
export PYTORCH_ROCM_ARCH=gfx900 
```

Fermez la connexion SSH courante, ouvrez en une autre, puis installez PyTorch comme suit :

```bash

(base) root@ubuntu-comfyui:~# cd /root
(base) root@ubuntu-comfyui:~# conda activate comfyenv

(comfyenv) root@ubuntu-comfyui:~# wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.3.2/torch-2.4.0%2Brocm6.3.2-cp312-cp312-linux_x86_64.whl

(comfyenv) root@ubuntu-comfyui:~# wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.3.2/torchvision-0.19.0%2Brocm6.3.2-cp312-cp312-linux_x86_64.whl

(comfyenv) root@ubuntu-comfyui:~# wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.3.2/pytorch_triton_rocm-3.0.0%2Brocm6.3.2.75cc27c26a-cp312-cp312-linux_x86_64.whl

(comfyenv) root@ubuntu-comfyui:~# wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.3.2/torchaudio-2.4.0%2Brocm6.3.2-cp312-cp312-linux_x86_64.whl

(comfyenv) root@ubuntu-comfyui:~# pip3 uninstall torch torchvision torchaudio pytorch-triton-rocm

(comfyenv) root@ubuntu-comfyui:~# pip3 install torch-2.4.0+rocm6.3.2-cp312-cp312-linux_x86_64.whl torchvision-0.19.0+rocm6.3.2-cp312-cp312-linux_x86_64.whl torchaudio-2.4.0+rocm6.3.2-cp312-cp312-linux_x86_64.whl pytorch_triton_rocm-3.0.0+rocm6.3.2.75cc27c26a-cp312-cp312-linux_x86_64.whl

(comfyenv) root@ubuntu-comfyui:~# rm *.whl
```

L'installation doit se dérouler sans échec.

Testez l'installation de PyTorch à l'aide du script ci-dessous :

```bash
(comfyenv) root@debian-comfyui:~# python3
>>> import torch
>>> print(torch.cuda.is_available())
>>> print(torch.cuda.get_device_name(torch.cuda.current_device()))
```

Les _print_ renvoient _True_ et le _nom du GPU_ situé sur l'hôte.  
CTRL+D pour quitter Python.  

Tests complémentaires :

```bash
(comfyenv) root@debian-comfyui:~# cd /root
(comfyenv) root@debian-comfyui:~# python3 -c 'import torch' 2> /dev/null && echo 'Success' || echo 'Failure'
```

Retour :

```markdown
Success
```

```bash
(comfyenv) root@debian-comfyui:~# pip show torch
```

Retour :

```markdown
Name: torch
Version: 2.4.0+rocm6.3.2
Summary: Tensors and Dynamic neural networks in Python with strong GPU acceleration
Home-page: https://pytorch.org/
Author: PyTorch Team
Author-email: packages@pytorch.org
License: BSD-3
Location: /root/miniconda3/envs/comfyenv/lib/python3.12/site-packages
Requires: filelock, fsspec, jinja2, networkx, pytorch-triton-rocm, setuptools, sympy, typing-extensions
Required-by: torchaudio, torchvision
```

#### _- Installation de ComfyUI_

Installez si besoin le paquet _git_ et clonez le dépot suivant :

```bash
(comfyenv) root@ubuntu-comfyui:~# cd /root
(comfyenv) root@ubuntu-comfyui:~# git clone https://github.com/comfyanonymous/ComfyUIls
```

Puis, installez ComfyUI comme suit :

```bash
(comfyenv) root@ubuntu-comfyui:~# cd /root/ComfyUI
(comfyenv) root@ubuntu-comfyui:~# pip3 install -r requirements.txt
```

Ajoutez ensuite l'utilisateur _root_ aux groupes _video_ et _render_.

```bash
root@ubuntu-comfyui:~# usermod -aG video root
root@ubuntu-comfyui:~# usermod -aG render root
```

Lancez enfin le serveur ComfyUI comme suit :

```bash
(comfyenv) root@ubuntu-comfyui:~# cd /root/ComfyUI
(comfyenv) root@ubuntu-comfyui:~# HSA_OVERRIDE_GFX_VERSION=9.0.0 python main.py --listen 0.0.0.0
```

et ouvrez depuis un navigateur Web local l'URL `http://ip-ubuntu-comfyui:8188`.

Redémarrez le conteneur depuis Proxmox et créez le fichier suivant qui sera utilisé par la suite pour démarrer automatiquement le serveur de ComfyUI :

```bash
(base) root@ubuntu-comfyui:~# cd /root
(base) root@ubuntu-comfyui:~# nano start_comfyui.sh
```

Entrez le contenu suivant :

```markdown
#!/bin/bash

# Activation de l'environnement conda de nom comfyenv
source /root/miniconda3/etc/profile.d/conda.sh
conda activate /root/miniconda3/envs/comfyenv

# Démarrage du serveur ComfyUI
export GIT_PYTHON_GIT_EXECUTABLE=/usr/bin/git
cd /root/ComfyUI
HSA_OVERRIDE_GFX_VERSION=9.0.0 python main.py --listen 0.0.0.0
```

Puis, rendez le fichier exécutable :

```bash
(base) root@ubuntu-comfyui:~# chmod +x start_comfyui.sh
```

Créez un fichier de service Systemd pour ComfyUI :

```bash
(base) root@ubuntu-comfyui:~# nano /etc/systemd/system/comfyui.service
```

et entrez le contenu suivant :

```markdown
[Unit]
Description=ComfyUI Service
After=network.target

[Service]
Environment=PATH=/root/miniconda3/bin
ExecStart=/root/start_comfyui.sh
WorkingDirectory=/root/ComfyUI
StandardOutput=inherit
StandardError=inherit
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

Démarrez le service :

```bash
(base) root@ubuntu-comfyui:~# systemctl daemon-reload
(base) root@ubuntu-comfyui:~# systemctl start comfyui
(base) root@ubuntu-comfyui:~# systemctl status comfyui
```

Si le statut est bon, validez le démarrage automatique :

```bash
(base) root@ubuntu-comfyui:~# systemctl enable comfyui
```

Pour finir, redémarrez le conteneur et testez de nouveau l'URL `http://ip-ubuntu-comfyui:8188`.

L'interface graphique affiche alors un flux de travail par défaut nommé _Unsave Workflow_.

#### _- ComfyUI Manager_

ComfyUI Manager offre une interface conviviale pour gérer les nœuds personnalisés, les modèles, les MAJ, etc...

Installez ce noeud personnalisé comme suit :

```bash
(base) root@ubuntu-comfyui:~# cd ComfyUI/custom_nodes
(base) root@ubuntu-comfyui:~# git clone https://github.com/ltdrdata/ComfyUI-Manager comfyui-manager
```

Puis, redémarrez le conteneur et vérifiez la présence de ComfyUI Manager dans la barre de menus de l'interface Web de ComfyUI.

Pour info, l'un de mes flux de travail a nécessité pour interagir avec les modèles d'Ollama d'installer depuis le ComfyUI Manager les éléments suivants :

Noeuds personnalisés :  
\- ComfyUI-Easy-Use  
\- ComfyUI-IF_AI_tools  
\- comfyui-ollama

Modèle :  
\- v1-5-pruned-emaonly.ckpt

#### _- Mise à jour de ComfyUI_

La mise à jour peut s'effectuer en cliquant directement sur le bouton _Update ComfyUI_ du ComfyUI Manager.

### Glances sur Proxmox

Glances peut afficher en temps réel l'utilisation des CPU et GPU, l'utilisation des mémoires système et GPU ainsi que les températures des uns et des autres, le tout sur une seule page accessible depuis un navigateur Web.

Avant de l'installer, vérifiez la présence sur l'hôte Proxmox des paquets _psutils_ et _lm-sensors_, à défaut ajoutez les.

Puis, procédez ainsi pour installer Glances :

```bash
proxmox:~# bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/glances.sh)"
```

Le même script peut-être lancé pour désinstaller Glances par la suite.

Une fois installé, ouvrez le port 61208 sur Proxmox et testez l'URL `http://ip-proxmox:61208`.

Testez la Cde suivante :

```bash
proxmox:~# glances -V
```

Celle-ci doit retourner le chemin du fichier _glances.log_.

Copiez le fichier /usr/local/share/doc/glances/glances.conf dans /etc/glances :

```bash
proxmox:~# mkdir /etc/glances
proxmox:~# cp /usr/local/share/doc/glances/glances.conf /etc/glances/
```

Editez ensuite la configuration :

```bash
proxmox:~# nano /etc/glances/glances.conf
```

et modifiez la ligne suivante :

```markdown hl_lines="2"
 # Available stats are: cpu,mem,load,swap
 list=cpu,mem,load
```

comme suit :

```markdown hl_lines="2"
 # Available stats are: cpu,mem,load,swap
 list=cpu,mem,load,swap
```

Redémarrez le service glances et observez le résulat dans la page Web de Glances.

```bash
proxmox:~# systemctl restart glances
```

Quelques remarques concernant le tableau de bord affiché :

\- Composite, Sensor 1,2 et 8 = Températures du dique nvme-pci  
\- Tctl = Température du CPU Ryzen _(k10temp-pci-00c3)_  
\- edge = temperature du GPU

### Remarques diverses

ComfyUI peut se connecter sur des modèles LLM d'Ollama comme Gemma, Llama et LLaVa en exploitant le noeud personnalisé ComfyUI-IF_AI_tools.

Ci-dessous, chemins du menu contextuel pour ajouter des noeuds personnalisés dans l'espace workflow de ComfyUI :

\- Ajout du noeud _IF Prompt Maker_ :  
=> Add Node => ImpactFrames => IF_tools  
=> IF Prompt Maker _(Remplace le IF Prompt to Prompt)_

Le noeud IF Prompt Maker est un outil de génération de prompts pour Stable Diffusion, permettant d'améliorer la composition et l'exactitude des prompts.

\- Ajout du noeud _Point de contrôle_ :  
=> Add Node => Chargeurs => Charger Point de Contrôle

Le noeud Point de contrôle est un modèle essentiel utilisé par Stable Diffusion pour générer des images.

\- Ajout du noeud _Charger VAE_ :  
=> Add Node => Chargeurs => Charger VAE

Le noeud VAE _(Variational AutoEncoder)_ encode les images en représentations latentes, permettant une compression efficace des caractéristiques visuelles.

\- Ajout du noeud _IF Save Text_ :  
=> Add Node => ImpactFrames => IF_tools => IF Save Text

Le noeud IF Save Text permet de sauvegarder du texte dans différents formats _(CSV, TXT, JSON)_ selon le mode de sauvegarde choisi.

\- Ajout du noeud _KSampler_ :  
=> Add Node => échantillonage => KSampler

Le noeud KSampler est essentiel pour l'échantillonnage dans la génération d'images, permettant d'affiner progressivement la qualité des images à travers plusieurs étapes.

\- Ajout du noeud _CLIP Text Encode_ :  
=> Add Node => conditionnement => CLIP Text Encode

Le nœud CLIPTextEncode encode les entrées textuelles en utilisant un modèle CLIP, facilitant la transformation du texte en vecteurs pour le conditionnement des modèles génératifs.

Options Clic droit sur le noeud :  
=> menu contextuel => Colors  
=> Convert Widget to Input => Convert text to input

\- Ajout du noeud _Image Latente Vide_ :  
=> Add Node => latent => Image Latente Vide

Le noeud Image Latente Vide sert de point de départ pour la génération et la manipulation d'images dans l'espace latent.

\- Ajout du noeud _VAE Decode_ :  
=> Add Node => latent => VAE Decode

Le nœud VAE Decode transforme les représentations latentes en images, permettant ainsi de visualiser les données encodées.

\- Ajout du noeud _Enregistrer Image_ :  
=> Add Node => image => Enregistrer Image

Le nœud Enregistrer Image permet d'enregistrer des images générées dans le dossier _ComfyUI/output/_.

D'autres noeuds peuvent être ajoutés dans le flux de travail permettant ainsi d'obtenir le résultat souhaité en matière de génération d'images depuis un texte.

**Fin.**
