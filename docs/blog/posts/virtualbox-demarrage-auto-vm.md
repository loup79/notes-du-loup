---
title: VirtualBox - Start auto des VM
summary: Démarrage automatique des VM sous Windows et Debain.
authors: 
  - G.Leloup
date: 2026-05-19
categories: 
  - 01 Hyperviseur Virtualbox
---

<figure markdown>
![Image - www.liblogo.com - Virtualbox Logo](../images/2026/01/virtualbox-demarrage-auto.webp)
</figure>

## Démarrage automatique des VM

Lancez vos VM automatiquement lors d'un redémarrage du système Windows ou Debian.

### - - Sous Windows 11 - -

VirtualBox est installé par défaut dans :

```txt
C:\Program Files\Oracle\VirtualBox\
```

et son fichier de configuration VirtualBox.xml est dans :

```txt
C:\Utilisateurs\nom-du-user\.VirtualBox\
```

#### _- Fichier autostart.properties_

Créez un fichier de nom autostart.properties dans le dossier \Utilisateurs\nom-du-user\\.VirtualBox\\ et remplissez-le avec le contenu suivant :

```markdown
# La politique par défaut est de refuser le démarrage  
# d'une VM "deny", l'autre est de l'autoriser "allow".
default_policy = deny

# L'utilisateur user-x ci-dessous est autorisé à démarrer  
# des VM mais ceci après un délai de 30 secondes
nom-du-user = {
     allow = true
     startup_delay = 30
}
# Saut de ligne obligatoire après l'acollade
```

Le service qui exploitera ce fichier s'appelle VirtualBox Autostart Service _(VBoxAutostartSvc)_.

#### _- Svc VBoxAutostartSvc_

Créez au préalable la variable d'environnement Windows de nom VBOXAUTOSTART_CONFIG.

La Cde ci-dessous peut être utilisée depuis le Terminal _(administrateur)_ de Windows pour créer temporairement celle-ci :

```txt
set VBOXAUTOSTART_CONFIG=C:\Users\nom-du-user\.VirtualBox\autostart.properties
```

<!-- more -->

Pour rendre cette variable permanente, utilisez l'application sysdm.cpl qui peut être lancée depuis le champ Exécuter du menu Windows.

Une fenêtre Propriétés système s'ouvre :  
-> Onglet Paramètres système avancés  
-> Bouton Variables d'environnement  
-> Zone Variables système  
-> Bouton Nouvelle...  

-> Champ Nom de la variable, entrez :  
VBOXAUTOSTART_CONFIG

-> Champ Valeur de la variable, entrez :  
C:\Users\nom-du-user\\.VirtualBox\autostart.properties

-> Bouton OK

La création de la variable _VBOXAUTOSTART_CONFIG_ peut être vérifiée depuis l'application regedit en accédant à la clé Environment ci-dessous :

```markdown
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment
```

Ensuite, activez le service de démarrage auto des VM depuis le Terminal _(administrateur)_ de Windows comme ceci :

```txt
cd "C:\Program Files\Oracle\VirtualBox"
.\VBoxAutostartSvc.exe install --user=nom-du-user
```

Le MDP de l'utilisateur nom-du-user sera demandé.

Pour info, le service peut être désactivé ainsi :

```txt
.\VBoxAutostartSvc.exe delete --user=nom-du-user
```

Finir en autorisant le démarrage automatique de chacune des VM _(VM arrêtée)_ :

```txt
cd "C:\Program Files\Oracle\VirtualBox"

.\VBoxManage.exe modifyvm "nom-de-la-vm" --autostart-enabled on --defaultfrontend headless --autostart-delay 30
```

L'option --defaultfrontend headless implique un mode de démarrage sans fenêtre d'affichage graphique, ceci dans le cas par exemple d'une utilisation d'un accès à distance sur la VM de type RDP ou VNC sur la VM.

Ajoutez quelques secondes d'écart entre chaque autostart-delay.

Une VM déclarée en autostart voit son fichier de configuration *.vbox situé dans le dossier de celle-ci contenir la ligne suivante :

```markdown
<Autostart enabled="true" delay="30" autostop="Disabled"/>
```

#### _- Svc VBoxSVC_

VirtualBox prend en charge l'exécution du service VBoxSVC dans la session 0 _(zéro)_ de Windows.

VBoxSVC fonctionne alors comme un service Windows normal et permet aux VM headless de continuer à fonctionner même si l'utilisateur nom-du-user ferme sa session Windows.

Pour activer le service, démarrez l'application regedit de Windows, puis :

-> HKEY_LOCAL_MACHINE\Software\Oracle\VirtualBox

Créez une clé REG_DWORD de nom VBoxSDS avec la valeur 1 et redémarrez Windows.

C'est terminé pour Windows.

### - - Sous Debian - -

#### _- Fichier vbox.cfg_

Créez le fichier _/etc/default/virtualbox_ :

```bash
sudo nano /etc/default/virtualbox
```

et ajoutez le contenu suivant :

```markdown
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/vbox.cfg
```

Créez ensuite le fichier vbox.cfg :

```bash
sudo nano /etc/vbox/vbox.cfg
```

et entrez l'un des 2 contenus suivants :

```markdown
default_policy = deny
nom-du-user = {
allow = true
startup_delay = 10
}
```

```markdown
default_policy=deny
nom-du-user={allow=true startup_delau=10}
```

!!! note "Nota"
    VirtualBox 7.1.x semble accepter uniquement la syntaxe du deuxième contenu.

Ajoutez l'utilisateur nom-du-user au groupe vboxusers :

```bash
sudo usermod -aG vboxusers nom-du-user
```

Affectez le dossier /etc/vbox/ au groupe vboxusers :

```bash
sudo chgrp vboxusers /etc/vbox
```

et autorisez le groupe à écrire dans le dossier :

```bash
sudo chmod g+w /etc/vbox
```

Redémarrez l'hôte Debian pour la prise en compte des permissions.

#### _- Svc vboxautostart-service_

Virtualbox fournit dans le dossier /lib/systemd/system/ un service appelé vboxautostart-service qui se chargera de lancer les VM paramétrées en démarrage auto.

Commencez par autoriser le démarrage auto des VM :

```bash
VBoxManage setproperty autostartdbpath /etc/vbox
```

Puis stoppez les VM et listez les noms de celles-ci :

```bash
VBoxManage list vms
```

Enfin, accédez aux dossiers des VM et entrez pour chacune de celles-ci la Cde suivante :

```bash
VBoxManage modifyvm "nom-de-la-vm" --autostart-enabled on --defaultfrontend headless --autostart-delay 30
```

Ajoutez quelques secondes d'écart entre chaque autostart-delay.

Relancez le système, vérifiez le bon démarrage automatique des VM et l'ajout dans le dossier /etc/vbox/ du fichier nom-du-user.start.

C'est terminé pour Debian.

<center>---------- Fin ----------</center>
