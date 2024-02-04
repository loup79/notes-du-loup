---
title: "Proxmox Bare Metal"
summary: Notes diverses sur Proxmox.
authors: 
  - G.Leloup
date: 2024-02-04
categories: 
  - Proxmox
---

## Proxmox 8.1

### RDP Copier/Coller

Penser à positionner la fonction _Presse-papiers_ à **oui** dans la configuration de _mRemoteNG_ pour chacune des VM Linux tournant sous Proxmox.

Rien à faire de particulier du côté du serveur _xrdp_ installé sur les VM.

### Gestion du pare-feu

Vérifier sur la machine Proxmox le statut du service _pve-firewall_ :

```bash
# systemctl status pve-firewall
```

Retour normal :

```markdown
● pve-firewall.service - Proxmox VE firewall
     Loaded: loaded (/lib/systemd/system/pve-firewall.service; enabled; preset: enabled)
     Active: active (running) since Mon 2024-01-08 11:28:53 CET; 3 weeks 6 days ago
   Main PID: 1185 (pve-firewall)
      Tasks: 1 (limit: 34638)
     Memory: 100.4M
        CPU: 6h 57min 55.895s
     CGroup: /system.slice/pve-firewall.service
             └─1185 pve-firewall

janv. 08 11:28:51 labos5 systemd[1]: Starting pve-firewall.service - Proxmox VE firewall...
janv. 08 11:28:53 labos5 pve-firewall[1185]: starting server
janv. 08 11:28:53 labos5 systemd[1]: Started pve-firewall.service - Proxmox VE firewall.
```

Vérifier ensuite son activation :

```bash
# pve-firewall status
```

Retour par défaut :

```markdown
Status: disabled/running
```

Cela signifie que le service est par défaut démarré mais non activé.

#### Niveau Centre de données

Onglet _Pare-feu_ du menu _Centre de données_ :  
Créer 3 règles de pare-feu pour autoriser les accès aux ports WEB, SSH et RDP de proxmox.

Sous-onglet _Options_ :  
Activer le pare-feu en mettant la valeur du champ _Pare-feu_ à **oui**.

Vérifier ensuite son activation :

```bash
# pve-firewall status
```

Retour :

```markdown
Status: enable/running
```

Conséquences liées à l'activation du pare-feu :  
Les pings vers l'IP du **noeud** Proxmox Bare Metal ne fonctionnent plus.  
Les pings vers les IP des VM/CTN fonctionnent toujours.

#### Niveau Noeud

Onglet _Pare-feu_ du menu _Noeud "lab"_ :  
Créer 1 règle pour autoriser les pings **uniquement** depuis une adresse IP du réseau local.

Sous-onglet _Options_ :  
Activer le pare-feu en mettant la valeur du champ _Pare-feu_ à **oui**.

Conséquences liées à l'activation :  
Les pings vers l'IP du **noeud** fonctionnent uniquement depuis l'IP filtrée.  
Idem pour les pings vers les IP des VM/CTN.

#### Niveau VM/CTN

Créer, selon la même méthode que ci-dessus, les règles de pare-feu qui conviennent pour chacune des VM/CTN.

### Nginx Proxy Manager

Lien utile :  
[proxmox-scripts](https://github.com/ej52/proxmox-scripts/tree/main/apps/nginx-proxy-manager){ target="_blank" }

#### Création du conteneur

Créer un conteneur LXC contenant l'application _Nginx Proxy Manager_ qui servira de _reverse proxy_ pour l'ensemble des VM et conteneurs de Proxmox.

Pour cela, se connecter en SSH sur le serveur Proxmox et entrer la Cde suivante :

```bash
# bash -c "$(wget --no-cache -qO- https://raw.githubusercontent.com/ej52/proxmox/main/create.sh)" -s --app nginx-proxy-manager --cleanup --os debian --os-version latest --hostname reverse-proxy
```

Le conteneur est créé automatiquement et reçoit une IP depuis le sereur DHCP du réseau local.

L'interface WEB de _Nginx Proxy Manager_ devient accessible depuis l'URL :  
`http://192.168.x.y:81`

Login et MDP par défaut :  
Login : `admin@example.com`  
MDP : changeme

#### Nom de domaine et DynDNS

Enregistrer un nom de domaine sur le site [Duck DNS](https://www.duckdns.org/){ target="_blank" }, Ex : `wxyz.duckdns.org`.

Vérifier ensuite sur Proxmox la présence des paquets _cron_ et _curl_ :

```bash
# apt list cron
# apt list curl
```

Les installer si manquants puis créer les dossiers et fichiers suivants :

```bash
# cd /root
# mkdir duckdns
# cd duckdns
# touch duck.sh
```

Editer le script _duck.sh_ et entrer le contenu suivant :

```markdown
echo url="https://www.duckdns.org/update?domains=nom-du-domaine&token=f30a8c59-b492-4323-...&ip=" | curl -k -o ~/duckdns/duck.log -K -
```

Modifier les permissions du script :

```bash
# chmod 700 /root/duckdns/duck.sh
```

Lancer enfin la Cde ci-dessous :

```bash
# crontab -e
```

et ajouter cette ligne en fin de fichier :

```markdown
*/5 * * * * /root/duckdns/duck.sh >/dev/null 2>&1
```

Le script _duck.sh_ sera exécuté toutes les 5 minutes afin que Duck DNS ait connaissance de l'adresse IP associée au nom de domaine `wxyz.duckdns.org`.

#### Certificat Let's Encrypt



**Fin.**
