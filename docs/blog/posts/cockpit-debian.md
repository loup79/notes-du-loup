---
title: "Cockpit / Debian"
summary: Cockpit permet au travers d’une interface Web de gérer localement ou à distance les systèmes Linux.
authors: 
  - G.Leloup
date: 2026-09-06
categories: 
  - 13 Cockpit
---

<figure markdown>
  ![Synoptique - Cockpit - Mémento 12.1](../images/2026/04/Cockpit.webp){ width="430" }
</figure>

## Mémento 12.1 - Cockpit

[Cockpit](https://cockpit-project.org){ target="_blank" } permettra de gérer à distance les systèmes Linux des VM du réseau virtuel, ceci au travers d'une interface Web en utilisant le nom de domaine loupvirtuel.fr.

La gestion des VM sera centralisée depuis un serveur Cockpit primaire installé sur la VM srvdmz.

### Accès sur le réseau virtuel

Le domaine loupvirtuel.fr n'étant pas public, vous ne pouvez pas accéder au site Web du réseau virtuel depuis Internet en tapant son URL dans le champ adresse d'un navigateur Web.

En revanche, vous pouvez simuler cet accès Internet depuis un PC situé sur votre réseau local.

Par exemple, pour un PC Windows :

\- Etape 1  
Entrez sur ce PC, comme dans le mémento Contrôle à distance au [§ Test de connexion](../posts/controle-distant-debian.md#test-connexion){ target="_blank" }, les 3 routes statiques permettant de joindre les VM du réseau local virtuel :

```bash
[C:\~] route add -p 192.168.2.0 mask 255.255.255.0 192.168.x.w

[C:\~] route add -p 192.168.3.0 mask 255.255.255.0 192.168.x.w

[C:\~] route add -p 192.168.4.0 mask 255.255.255.0 192.168.x.w
```

192.168.x.w est l'IP de la carte réseau RED d'IPFire.  
Le -p déclare les routes comme étant permanentes.

\- Etape 2  
Ajoutez ensuite ces 2 lignes au fichier DNS C:\Windows\System32\drivers\etc\hosts :

```markdown
# loupvirtuel.fr
192.168.4.2 loupvirtuel.fr
```

\- Etape 3  
Finissez en testant l'URL `https://loupvirtuel.fr` :

<!-- more -->

<figure markdown>
  ![Capture - Site Web : Accueil du domaine loupvirtuel.fr](../images/2026/04/srvdmz-dotclear.webp){ width="580" }
  <figcaption>Site Web : Accueil du domaine loupvirtuel.fr</figcaption>
</figure>

Voilà, vous êtes prêt pour installer et utiliser Cockpit.

### Cockpit primaire sur srvdmz

Cockpit permet l'administration de son système Linux _(Cockpit primaire)_ ainsi que l'administration centralisée d'autres systèmes Linux _(Cockpits secondaires)_.

Les onglets de son interface Web proposent de :

-- Partie Système --

* Visualiser l'état du matériel -> _Aperçu_  
* Lire les journaux système -> _Journaux_  
* Visualiser les disques -> _Stockage_  
* Visualiser le trafic réseau -> _Réseau_  
* Gérer les comptes utilisateurs -> _Comptes_  
* Gérer les services -> _Services_

-- Partie Outils --

* Gérer les applications -> _Applications_  
* Gérer les MAJ -> _Mises à jour de logiciel_  
* Travailler avec un Terminal Web -> _Terminal_

Il est également possible, depuis cette interface, de :

* Gérer d'autres serveurs Linux _(centralisation)_  
* Gérer, créer des VM _(plugin cockpit-machines)_  
* Gérer des CTN Podman _(plugin cockpit-podman)_  
* Etc…

Pour installer Cockpit sur la VM srvdmz, entrez cette Cde :

-- **Debian 12** --

```bash
[srvdmz@srvdmz] sudo apt install cockpit cockpit-pcp
```

-- **Debian 13** --

```bash
[srvdmz@srvdmz] sudo apt install cockpit
```

### Réglages Port/Pare-feu/SSL

#### _- Port utilisé par Cockpit_

Le numéro de port par défaut de Cockpit est le 9090. Vous allez, par sécurité, modifier celui-ci.

Commencez par créer un dossier cockpit.socket.d :

```bash
[srvdmz@srvdmz] cd /etc/systemd/system/
[srvdmz@srvdmz] sudo mkdir cockpit.socket.d
```

Puis générez dans celui-ci un fichier listen.conf :

```bash
[srvdmz@srvdmz] cd cockpit.socket.d
[srvdmz@srvdmz] sudo nano listen.conf 
```

et entrez ceci pour déclarer l'usage du port 9528 :

```markdown
[Socket]
ListenStream=
ListenStream=9528
```

Pour finir, rechargez la nouvelle configuration systemd et relancez la partie réseau de Cockpit :

```bash
[srvdmz@srvdmz] sudo systemctl daemon-reload
[srvdmz@srvdmz] sudo systemctl restart cockpit.socket 
```

Le serveur Cockpit de la VM srvdmz écoutera ainsi sur le port 9528.

#### _- Pare-feu VM IPFire_

Autorisez l'usage du port 9528 au niveau de la VM IPFire _(Ref: Mémento [DNS split](../posts/dns-split-debian.md#pare-feu){ target="_blank" })_ :

<figure markdown>
  ![Capture - IPFire : Utilisation du port 9528 autorisé](../images/2026/04/ipfire-cockpit.webp){ width="580" }
  <figcaption>IPFire : Utilisation du port 9528 autorisé</figcaption>
</figure>

#### _- SSL domaine loupvirtuel.fr_

Cockpit fournit de base sous Debian 12 _(Pas sous Debian 13)_ un certificat SSL auto-signé pour les connexions HTTPS entrantes, une alerte de sécurité est alors affichée par les navigateurs Web.

Vous allez, pour éviter cela, utiliser le certificat du domaine loupvirtuel.fr _(Ref : [§ Protocole HTTPS](../posts/lamp-https-cms-partie-2-debian.md#https){ target="_blank" } du mémento LAMP HTTPS CMS : Partie 2)_.

Commencez par copier ces 2 fichiers SSL dans le dossier /etc/cockpit/ws-certs.d :

```bash
[srvdmz@srvdmz] cd /etc/ssl
  
[srvdmz@srvdmz] sudo cp loupvirtuel.crt /etc/cockpit/ws-certs.d 
  
[srvdmz@srvdmz] sudo cp loupvirtuel.key /etc/cockpit/ws-certs.d
```

Renommez, si présent dans le dossier ci-dessous, le certificat auto-signé de Cockpit :

```bash
[srvdmz@srvdmz] cd /etc/cockpit/ws-certs.d
  
[srvdmz@srvdmz] sudo mv 0-self-signed.cert 0-self-signed.cert-save
```

Puis si sous **Debian 12**, modifiez les droits sur les 2 fichiers importés comme suit :

```bash
[srvdmz@srvdmz] sudo chown root:cockpit-ws loupvirtuel.*
```

Sous **Debian 13**, les permissions d'accès aux fichiers peuvent rester à root:root.

Redémarrez Cockpit :

```bash
[srvdmz@srvdmz] sudo systemctl restart cockpit
[srvdmz@srvdmz] sudo systemctl status cockpit
```

Cockpit est maintenant accessible localement depuis l'URL :  
`https://loupvirtuel.fr:9528`

<figure markdown>
  ![Capture - Cockpit : Fenêtre de login](../images/2026/04/srvdmz-cockpit-login.webp){ width="580" }
  <figcaption>Cockpit : Fenêtre de login</figcaption>
</figure>

Connectez-vous en tant qu'utilisateur srvdmz.

La page d'accueil de Cockpit doit s'afficher :  
-> Bouton _Activez l'accès administrateur (Turn on ... access)_

Une fenêtre _Passer à l’accès administrateur (Switch to ...)_ s'ouvre :  
-> Mot de passe de srvdmz : Entrez le MDP de srvdmz  
-> Bouton _S'authentifier (Authenticate)_

<figure markdown>
  ![Capture - Cockpit : Accueil = Onglet Aperçu](../images/2026/04/srvdmz-cockpit-accueil.webp){ width="580" }
  <figcaption>Cockpit : Accueil = Onglet Aperçu</figcaption>
</figure>

Le contenu de la zone de notification, soit celui du fichier /etc/motd, peut maintenant être modifié en cliquant sur l'icône d'édition située dans la zone.

Accédez mensuite au Widget Utilisation :  
-> Voir les métriques et l'historique

Cliquez sur le bouton _Installer la prise en charge PCP_, les métriques apparaîtront au bout de quelques minutes.

!!! note "Nota"
    Si vous disposez d'un nom de domaine routé par votre Box Internet sur l'un de vos serveurs, vous pouvez utiliser celui-ci pour joindre depuis Internet le Cockpit de la VM srvdmz, ceci en créant une règle de proxy inverse appropriée.

### Interface Web de Cockpit

Observez maintenant le contenu de chacun des modules de Cockpit, soit :

-- Partie Système --

* Journaux _(Priorité = Erreur et au dessus pour ...)_

<figure markdown>
  ![Capture - Cockpit : Onglet Journaux](../images/2026/04/srvdmz-cockpit-journaux.webp){ width="580" }
  <figcaption>Cockpit : Onglet Journaux</figcaption>
</figure>

Contrôle des logs du système, options de filtrage.

* Stockage

<figure markdown>
  ![Capture - Cockpit : Onglet Stockage](../images/2026/04/srvdmz-cockpit-stockage.webp){ width="580" }
  <figcaption>Cockpit : Onglet Stockage</figcaption>
</figure>

Gestion du stockage, options de gestion RAID et NFS.

* Réseau

<figure markdown>
  ![Capture - Cockpit : Onglet Réseau](../images/2026/04/srvdmz-cockpit-reseau.webp){ width="580" }
  <figcaption>Cockpit : Onglet Réseau</figcaption>
</figure>

Gestion du réseau, options d'ajout de Lien/Pont/VLAN, etc...

* Comptes

<figure markdown>
  ![Capture - Cockpit : Onglet Comptes](../images/2026/04/srvdmz-cockpit-comptes.webp){ width="580" }
  <figcaption>Cockpit : Onglet Comptes</figcaption>
</figure>

Gestion des comptes, options de création Groupe/Utilisateur.

* Services

<figure markdown>
  ![Capture - Cockpit : Onglet Services](../images/2026/04/srvdmz-cockpit-services.webp){ width="580" }
  <figcaption>Cockpit : Onglet Services</figcaption>
</figure>

Gestion des statuts des services, options de filtrage.

-- Partie Outils --

* Applications

<figure markdown>
  ![Capture - Cockpit : Onglet Applications](../images/2026/04/srvdmz-cockpit-applications.webp){ width="580" }
  <figcaption>Cockpit : Onglet Applications</figcaption>
</figure>

Gestion des extensions utilisées _(plugins)_.

* Mises à jour logicielles

<figure markdown>
  ![Capture - Cockpit : Onglet Mises à jour de logiciel](../images/2026/04/srvdmz-cockpit-maj.webp){ width="580" }
  <figcaption>Cockpit : Onglet Mises à jour de logiciel</figcaption>
</figure>

MAJ des paquets du système, option de redémarrage.

* Terminal _(très pratique)_

<figure markdown>
  ![Capture - Cockpit : Onglet Terminal](../images/2026/04/srvdmz-cockpit-terminal.webp){ width="580" }
  <figcaption>Cockpit : Onglet Terminal</figcaption>
</figure>

Usage de la ligne de Cde pour administrer le système.

La déconnexion de Cockpit s'effectue depuis le menu Session situé en haut et à droite de la page Web.

### Gestion centralisée du réseau

Il est possible de gérer l'ensemble des VM et CTN Podman du réseau local virtuel depuis le Cockpit installé sur la VM srvdmz.

Pour cela, il faut rendre les VM à gérer accessibles en protocole SSH _(Ref: Mémento [SSH sur VM ovs (OpenvSwitch)](../posts/controle-distant-debian.md#ssh-ovs){ target="_blank" })_ et installer Cockpit sur celles-ci.

SSH ne propose pour l'instant que la lecture seule sur les VM distantes, il faudra toujours cliquer sur le bouton _Activez l'accès administrateur_ de celles-ci pour les gérer.

SSH augmentera la sécurité entre les VM et évitera d'avoir à saisir les MDP de façon répétée.

Les VM à gérer depuis le Cockpit de la VM srvdmz seront à créer en tant que nouveaux hôtes.

#### _- Ajout de srvlan (nouvel hôte)_

Commencez par installer un serveur SSH sur la VM srvlan :

```bash
[srvlan@srvlan] sudo apt install openssh-server
[srvlan@srvlan] sudo systemctl status sshd
```

Editez ensuite le fichier de configuration du serveur :

```bash
[srvlan@srvlan] sudo nano /etc/ssh/sshd_config
```

et remplacez la ligne #Port 22 par Port 222.

Relancez le service SSH pour traiter la modification :

```bash
[srvlan@srvlan] sudo systemctl restart sshd
```

et installez Cockpit sur la VM srvlan comme pratiqué sur la VM srvdmz soit :

```bash
[srvlan@srvlan] sudo apt install cockpit
```

-- **Debian 12** --  
Connectez-vous ensuite sur le Cockpit de srvdmz et cliquez sur le menu déroulant situé en haut et à gauche de la page Web :  
-> Bouton _Ajouter un nouvel hôte_

Une fenêtre _Ajouter un nouvel hôte_ s'ouvre :  
-> Hôte : srvlan.intra.loupvirtuel.fr:222  
-> Nom d'utilisateur : srvlan  
-> Couleur : Choisissez une couleur pour la VM  
-> Bouton _Ajouter_

Une fenêtre _Nouvel hôte_ s'ouvre :  
-> Bouton Accepter la clé et se connecter

Une fenêtre _Connectez-vous à `srvlan@...`_ s'ouvre :  
-> Mot de passe : MDP de srvlan  
-> Cochez _Créer une nouvelle clé SSH et l'autoriser_

Le fenêtre ouverte s'étend :  
-> Mot de passe clé : MDP de srvdmz et non de srvlan  
-> Confirmer ... de passe de la clé : MDP de srvdmz  
-> Bouton _Connexion_

L'hôte `srvlan@...` est ajouté dans le menu déroulant situé en haut et à gauche de la page Web.

Fermez la connexion courante depuis le menu Session situé en haut et à droite de la page Web.

Fin Debian 12

-- **Debian 13** --  
Dans les versions récentes de Cockpit, la gestion multi-hôtes n'est plus activée par défaut, il faut créer un fichier cockpit.conf :

```bash
[srvdmz@srvdmz] sudo nano /etc/cockpit/cockpit.conf
```

y entrer le contenu suivant :

```markdown
[WebService]
AllowMultiHost=true
```

et redémarrer Cockpit pour profiter de cette gestion :

```bash
[srvdmz@srvdmz] sudo systemctl restart cockpit
```

Connectez-vous ensuite sur le Cockpit de srvdmz et cliquez sur le menu déroulant situé en haut et à gauche de la page Web :  
-> Bouton _Ajouter un nouvel hôte_

Une fenêtre _Ajouter un nouvel hôte_ s'ouvre :  
-> Hôte : srvlan.intra.loupvirtuel.fr:222  
-> Nom d'utilisateur : srvlan  
-> Couleur : Choisissez une couleur pour la VM  
-> Bouton _Ajouter_

Une fenêtre _Connecter à srvlan.intra.loupvirtuel.fr ?_ s'ouvre :  
-> Bouton _Connecter_

Une fenêtre _Nouvel hôte: srvlan.intra.loupvirtuel.fr_ s'ouvre :  
-> Bouton _Faire confiance à l'hôte et l'ajouter_

Une fenêtre _Connectez-vous à `srvlan@srvlan.intra`..._ s'ouvre :  
-> Mot de passe : MDP de srvlan  
-> Connexion automatique : Cochez _Créer ... clé SSH et l'autoriser_  
-> Mot de passe clé : MDP de srvdmz et non de srvlan  
-> Confirmer le mot de passe de la clé : MDP de srvdmz  
-> Bouton _Connexion_

La connexion sur la VM srvlan s'établit et l'hôte est ajouté dans le menu déroulant situé en haut et à gauche de la page Web.

Fermez la connexion courante depuis le menu Session situé en haut et à droite de la page Web.

Fin Debian 13

Remarques diverses :  
\- Un hôte srvlan.intra.loup... a été ajouté dans le fichier /home/srvdmz/.ssh/known_hosts.

\- 2 clés SSH id_rsa et id_rsa.pub ont été créées sur srvdmz dans /home/srvdmz/.ssh/.

\- La clé id_rsa.pub a été copiée comme authorized_keys sur srvlan dans /home/srvlan/.ssh/.

\- Sous **Debian 13** le nouvel hôte srvlan apparaît dans le fichier /etc/cockpit/machines.d/99-webui.json de la VM srvdmz.

Reconnectez-vous maintenant sur le Cockpit de la VM srvdmz et sélectionnez ensuite l'hôte srvlan, la liaison sera directement établie sans demande de MDP.

<figure markdown>
  ![Capture - Cockpit : Gestion de srvlan depuis srvdmz](../images/2026/04/cockpit-ajout-hote-srvlan.webp){ width="580" }
  <figcaption>Cockpit : Gestion de srvlan depuis srvdmz</figcaption>
</figure>

Les connexions futures depuis le Cockpit de la VM srvdmz sur celui de la VM srvlan se feront sans demande de MDP, ceci grâce à la clé SSH.

#### _- Ajout d'ovs (nouvel hôte)_

Un serveur SSH écoute déjà sur le port 222.

Installez seulement Cockpit avec la même Cde que celle utilisée sur les VM srvdmz et srvlan.

!!! note "Nota"
    Ajoutez le paquet cockpit-podman qui permettra de gérer les conteneurs actifs sur la VM ovs.

Connectez-vous enfin sur le Cockpit de la VM srvdmz et cliquez sur le menu déroulant situé en haut et à gauche de la page Web :  
-> Bouton _Ajouter un nouvel hôte_

-- **Exemple pour Debian 13** --
Une fenêtre _Ajouter un nouvel hôte_ s'ouvre :  
-> Hôte : ovs.intra.loupvirtuel.fr:222  
-> Nom d'utilisateur : switch  
-> Couleur : Choisissez une couleur pour la VM  
-> Bouton _Ajouter_

Une fenêtre _Connecter à ovs.intra.loupvirtuel.fr ?_ s'ouvre :  
-> Bouton _Connecter_

Une fenêtre _Nouvel hôte: ovs.intra.loupvirtuel.fr_ s'ouvre :  
-> Bouton _Faire confiance à l'hôte et l'ajouter_

Une fenêtre _Connectez-vous à `switch@ovs.intra...`_ s'ouvre :  
-> Mot de passe : MDP de switch  
-> Connexion automatique : Cochez cette fois _Autoriser la clé SSH_  
-> Bouton _Connexion_

La connexion sur la VM ovs s'établit et l'hôte est ajouté dans le menu déroulant situé en haut et à gauche de la page Web.

Vérifiez ensuite le contenu du fichier 99-webui.json et modifiez le comme suit si l'hôte ovs n'est pas présent :

```markdown hl_lines="8-9"
{
  "srvlan.intra.loupvirtuel.fr": {
    "visible": true,
    "color": "#ffbe6f",
    "address": "srvlan.intra.loupvirtuel.fr",
    "user": "srvlan",
    "port": "222"
  },
  "ovs.intra.loupvirtuel.fr": {
    "visible": true,
    "color": "#ff0000",
    "address": "ovs.intra.loupvirtuel.fr",
    "user": "switch",
    "port": "222"
  }
}
```

Fermez enfin la connexion courante depuis le menu Session situé en haut et à droite de la page Web.

L' hôte ovs a été ajouté dans le fichier /home/srvdmz/.ssh/known_hosts.

La clé SSH publique id_rsa.pub de la VM srvdmz a été copiée comme authorized_keys sur la VM ovs.

Les connexions futures sur l'hôte switch s'établiront directement sans demande de MDP.

!!! note "Nota"
    Cockpit affiche les conteneurs Podman rootless et rootfull sous réserve d'être en Accès administrateur.

#### - Accès Cockpit secondaire

Au préalable, ajoutez Cockpit et un serveur SSH sur les VM debian-vm*.

L'accès direct sur un Cockpit secondaire se fera au travers du Cockpit primaire de la VM srvdmz.

Pour, par exemple, joindre le Cockpit de debian-vm1 :  
-> Accédez à la fenêtre de login du Cockpit de la VM srvdmz  
-> Remplissez ensuite les champs comme ci-dessous

<figure markdown>
  ![Capture - Cockpit :  Liaison directe sur un Cockpit secondaire](../images/2026/04/cockpit-login-direct-cockpit-secondaire.webp){ width="430" }
  <figcaption>Cockpit : Liaison directe sur un Cockpit secondaire</figcaption>
</figure>

-> Cliquez sur le bouton _Connexion_

Une fenêtre _Nouvel hôte_ s'ouvre :  
-> Bouton _Accepter la clé et se connecter_

La connexion est établie, l'acceptation de la clé SSH ne sera plus demandée à l'avenir.

### Bilan

Cockpit est léger, fiable, sécurisé et son interface Web responsive permet d'administrer un système Linux facilement y compris depuis un smartphone.

La gestion centralisée de plusieurs systèmes Linux est un plus.

Il propose aussi un peu de supervision, ceci en affichant quelques métriques bien utiles sous forme graphique.

**Fin.**
