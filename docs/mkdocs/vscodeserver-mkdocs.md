---
title: MkDocs - VS Code Server
summary: Installation de VS Code Server sur Debian.
author: G.Leloup
date: 2023-06-01
---

## VS Code Server sur Debian

Tuto pour installer VS Code Server sur une VM Debian.

L'outil servira notamment à créer la documentation ==Markdown== du projet MkDocs.

### Installation ou MAJ

Créer le dossier suivant :

```bash
cd /home/user/Documents
mkdir code-server
```

et installer _VS Code Server_ comme suit :

```bash
cd code-server
curl -fOL https://github.com/coder/code-server/releases/download/v4.x.0/code-server_4.x.0_amd64.deb
sudo dpkg -i code-server_4.x.0_amd64.deb
sudo systemctl enable --now code-server@user
```

Dans le cas d'une MAJ, la version précédente sera remplacée automatiquement.

Vérifier le statut du serveur :

```bash
sudo systemctl status code-server@user
```

Les dossiers et fichiers de _VS Code Server_ ont été installés dans :  
_/usr/lib/code-server/_  
_/usr/lib/systemd/_  
_/home/user/.config/_  
_/home/user/.local/share/_

### Accès au serveur

Le serveur est accessible depuis l'URL :  
`http://127.0.0.1:8080`

Le MDP du login d'accès au serveur se trouve dans le fichier :  
/home/user/.config/code-server/config.yaml

Penser, en cas de problème de page blanche suite à un login, à redémarrer code-server.

```bash
sudo systemctl restart code-server@user
```

### Suppression du serveur

Pour supprimer si nécessaire celui-ci :

```bash
cd /home/user
rm -rf /home/user/.local/share/code-server /home/user/.config/code-server
sudo apt remove code-server
```

**Fin.**
