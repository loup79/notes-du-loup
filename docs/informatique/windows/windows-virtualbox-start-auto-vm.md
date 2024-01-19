---
title: Windows - VBox start auto VM
summary: Démarrage automatique des VM sous Windows.
author: G.Leloup
date: 2024-01-19
---

## VirtualBox - Démarrage auto des VM

### Partie 1

Créer un fichier de nom _autostart.properties_ dans le dossier _/Utilisateurs/user/.VirtualBox_.

Entrer ensuite le contenu suivant :

```bash
# La politique par défaut est de refuser "deny" le démarrage d'une  
# machine virtuelle,l'autre option est de l'autoriser "allow".
default_policy = deny

# L'utilisateur ci-dessous est autorisé à démarrer les machines  
# virtuelles mais ceci après un délai de 30 secondes
user = {
     allow = true
     startup_delay = 30
}
# Saut de ligne obligatoire après l'acollade
```

Le service dans windows exécutant ce script s'appelle _VirtualBox Autostart Service_.

### Partie 2

Ajouter au préalable une variable d'environnement de nom _VBOXAUTOSTART_CONFIG_.

Cde temporaire à utiliser depuis le Terminal(administrateur) de Windows :

```bash
set VBOXAUTOSTART_CONFIG=C:\Users\user\.VirtualBox\autostart.properties
```

Pour rendre la variable permanente, passer par l'outil graphique _sysdm.cpl_ qui peut être lanccé depuis le champ _Exécuter_ du menu Windows.

Une fenêtre Propriétés système s'ouvre :  
-> Onglet _Paramètres système avancés_  
-> Bouton _Variables d'environnement_  
-> Zone _Variables système_  
-> Bouton _Nouvelle..._  

Entrer _VBOXAUTOSTART_CONFIG_ dans le champ _Nom de la variable_.  
Entrer _C:\Users\user\.VirtualBox\autostart.properties_ dans le champ _Valeur de la variable_.

-> Bouton _OK_

Si bien fait, la valeur de la variable VBOXAUTOSTART_CONFIG peut être lue dans l'outil _regedit_.

Ensuite activer le service de VirtualBox comme suit depuis le Terminal(administrateur) et non PowerShell :

```bash
cd "C:\Program Files\Oracle\VirtualBox"
.\VBoxAutostartSvc.exe install --user=user
```

Le mot de passe du user sera demandé.

Puis autoriser le démarrage automatique de chacune des VM _(VM arrêtée)_ :

```bash
.\VBoxManage.exe modifyvm "nom-de-la-vm" --autostart-enabled on --defaultfrontend headless --autostart-delay 30
```

Prévoir 60 secondes d'écart entre chaque démarrage de VM.

**Fin**.
