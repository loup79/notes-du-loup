---
title: MkDocs - VS Code Server
summary: Installation de VS Code Server sur Debian.
author: G.Leloup
date: 2025-01-02
---

## VS Code Server sur Debian

Tuto pour installer VS Code Server sur une VM Debian.

L'outil servira notamment à créer la documentation ==Markdown== du projet MkDocs.

### Installation/MAJ du serveur

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

### Accès Web au serveur

Le serveur est accessible depuis l'URL :  
`http://127.0.0.1:8080`

Le MDP du login d'accès au serveur se trouve dans le fichier :  
/home/user/.config/code-server/config.yaml

Penser, en cas de problème de page blanche suite à un login, à redémarrer code-server.

```bash
sudo systemctl restart code-server@user
```

### Extensions ajoutées

French Language Pack for Visual Studio Code  
Jupyter  (Editeur Web Python)
Markdown All in One (Utilitaire Markdown)  
Python  (Langage de programmation)

### Accès au projet MkDocs

Cliquer sur l'icône EXPLORATEUR du menu de VS Code Server :  
-> Bouton _Ouvrir le dossier_  
-> Sélectionner le dossier _MkDocs_ de nom _notes-user_  
-> Bouton _OK_  
-> Bouton _Oui, je fais confiance aux auteurs_

Il ne reste plus qu'à créer la documentation.

### Hachage du MDP Web

Générer un MDP haché sans l'outil _npx_ :

```bash
cd /home/user/.config/code-server/
apt install argon2
export SALT="un-contenu-au-hazard"
echo -n "mdp-en-clair" | argon2 $SALT -e
```

Exemple de retour de la Cde _echo_ :

```markdown
$argon2i$v=19$m=4096,t=3,p=1$bGup8sbW45...
```

Modifier ensuite le fichier _config.yaml_ en remplaçant _password:_ par :

```markdown
hashed-password: "$argon2i$v=19$m=4096,t=3,p=1$bGup8sbW45..."
```

Pour finir, redémarrer _code-server_ :

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
