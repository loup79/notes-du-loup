---
title: VirtualBox - Start auto des VM
summary: Démarrage automatique des VM sous Windows et Debain.
authors: 
  - G.Leloup
date: 2024-01-20
categories: 
  - 1 - Hyperviseur Virtualbox
---

## Démarrage automatique des VM

### - - Sous Windows - -

VirtualBox est installé par défaut dans :

```bash
C:\Program Files\Oracle\VirtualBox\
```

et son fichier de configuration _VirtualBox.xml_ est dans :

```bash
C:\Utilisateurs\user\.VirtualBox\
```

#### _Fichier autostart.properties_

Créez un fichier de nom _autostart.properties_ dans le dossier _/Utilisateurs/user/.VirtualBox_ et remplissez-le avec le contenu suivant :

```bash
# La politique par défaut est de refuser "deny" 
# le démarrage d'une machine virtuelle,l'autre
# option est de l'autoriser "allow".
default_policy = deny

# L'utilisateur ci-dessous est autorisé à
# démarrer les machines virtuelles mais ceci 
# après un délai de 30 secondes
user = {
     allow = true
     startup_delay = 30
}
# Saut de ligne obligatoire après l'acollade
```

Le service qui exploitera ce fichier s'appelle _VirtualBox Autostart Service_.

#### _Service VBoxAutostartSvc_

Créez au préalable une variable d'environnement Windows de nom _VBOXAUTOSTART_CONFIG_.

Cde pouvant être utilisée depuis le _Terminal(administrateur)_ de Windows pour créer temporairement celle-ci :

```bash
> set VBOXAUTOSTART_CONFIG=C:\Users\user\.VirtualBox\autostart.properties
```

<!-- more -->

Pour rendre cette variable permanente, utilisez l'application _sysdm.cpl_ qui peut être lanccé depuis le champ _Exécuter_ du menu Windows.

Une fenêtre _Propriétés système_ s'ouvre :  
-> Onglet _Paramètres système avancés_  
-> Bouton _Variables d'environnement_  
-> Zone _Variables système_  
-> Bouton _Nouvelle..._  

-> Champ _Nom de la variable_, entrez :  
_VBOXAUTOSTART_CONFIG_.

-> Champ _Valeur de la variable_, entrez :  
_C:\Users\user\\.VirtualBox\autostart.properties_.

-> Bouton _OK_

La création de la variable _VBOXAUTOSTART_CONFIG_ peut être vérifiée depuis l'application _regedit_ en accédant à la clé ci-dessous :

```bash
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment
```

Ensuite activez le service de démarrage automatique des VM comme suit depuis le _Terminal(administrateur)_ de Windows :

```bash
> cd "C:\Program Files\Oracle\VirtualBox"
> .\VBoxAutostartSvc.exe install --user=user
```

Le MDP du l'utilisateur _user_ sera demandé.

Pour info, le service peut par la suite être désactivé ainsi :

```bash
> .\VBoxAutostartSvc delete --user=user
```

Finir en autorisant le démarrage automatique de chacune des VM _(VM arrêtée)_ :

```bash
> cd "C:\Program Files\Oracle\VirtualBox"
> .\VBoxManage.exe modifyvm "nom-de-la-vm" --autostart-enabled on --defaultfrontend headless --autostart-delay 30
```

L'option _--defaultfrontend headless_ implique un mode de démarrage _sans fenêtre d'affichage graphique_, ceci dans le cas d'une utilisation d'un accès à distance sur la VM de type RDP ou VNC.

Prévoir 60 secondes d'écart entre chaque démarrage de VM.

Une VM déclarée en autostart voit son fichier de configuration _*.vbox_ situé dans le dossier de celle-ci contenir la ligne suivante :

```bash
<Autostart enabled="true" delay="90" autostop="Disabled"/>
```

### - - Sous Linux - -

#### _Partie 1_

**Fin**.
