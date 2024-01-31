---
title: MkDocs - VS Code Server
summary: Installation de VS Code Server sur Debian.
author: G.Leloup
date: 2023-06-01
---

## VS Code Server sur Debian

Tuto pour installer VS Code Server sur une VM Debian.

L'outil servira notamment à créer la documentation Markdown du projet MkDocs.

### Installation ou MAJ

Créer le dossier suivant :

```bash
cd /home/user/Documents
mkdir code-server
```

et installer _VS Code Server_ comme suit :

```bash
curl -fOL https://github.com/coder/code-server/releases/download/v4.x.0/code-server_4.x.0_amd64.deb
sudo dpkg -i code-server_4.x.0_amd64.deb
sudo systemctl enable --now code-server@user
```

Retour :

```markdown
mkdocs, version 1.5.3 from /home/user.../site-packages/mkdocs (Python 3.11)
```



**Fin.**
