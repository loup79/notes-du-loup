---
title: "Proxmox - Nginx Proxy Manager"
summary: Notes diverses sur Proxmox.
authors: 
  - G.Leloup
date: 2024-02-04
categories: 
  - Proxmox
---

## Proxmox 8.1

### RDP Copier/Coller

Penser à positionner la fonction _Presse-papiers_ à **oui** dans la configuration de _mRemoteNG_ pour les VM tournant sous Proxmox.

Rien à faire de particulier du côté des serveurs _xrdp_ installés sur les VM.

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

Contrôler ensuite l'activation du service :

```bash
# pve-firewall status
```

Retour par défaut :

<!-- more -->

```markdown
Status: disabled/running
```

On peut voir que le service est par défaut démarré mais non activé.

#### Niveau Centre de données

Onglet _Pare-feu_ du menu _Centre de données_ :  
Créer 3 règles de pare-feu pour autoriser les accès aux ports WEB, SSH et RDP de Proxmox.

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

### Sécurité complémentaire

Interdire la connexion SSH en tant que _root_ et modifier le numéro de port utilisé par défaut.

Pour cela, éditer le fichier _/etc/ssh/sshd_config et modifier ces 2 lignes comme ceci :

```markdown
Port nouveau-numero-port
PermitRootLogin no
```

Dans le cas d'une authentification SSH par clé publique et non par MDP, ajouter ceci :

```markdown
PasswordAuthentication no
PubkeyAuthentication yes
```

### Sauvegarde des conteneurs

Il est nécessaire pour éviter un échec de modifier le fichier de configuration _/etc/vzdump.conf_.

Remplacer la ligne :

```markdown
tmpdir: DIR
```

par :

```markdown
tmpdir: /tmp
```

### Nginx Proxy Manager

Lien utile :  
[proxmox-scripts](https://community-scripts.github.io/ProxmoxVE/){ target="_blank" }

#### Création du conteneur

Le conteneur LXC contenant l'application _Nginx Proxy Manager_ servira de _reverse proxy_ pour l'ensemble des VM/CTN de Proxmox.

Se connecter en tant que _root_ et en SSH sur le serveur Proxmox puis entrer la Cde suivante :

```bash
# bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/nginxproxymanager.sh)"
```

Une fenêtre s'ouvre, entrer dans le mode avancé et configurer les paramètres proposés.

Une fois fait, le conteneur est créé automatiquement.

La Cde ci-dessus peut être réutilisée à l'intérieur du conteneur créé pour effectuer les MAJ.

L'interface WEB de _Nginx Proxy Manager_ devient accessible depuis l'URL :  
`http://192.168.x.y:81`

Login et MDP par défaut :  
Login : `admin@example.com`  
MDP : changeme

Commencer par changer le Login et MDP une fois rentré dans l'interface WEB.

Vérifier par curiosité le statut du service _Nginx Proxy Manager_ :

```bash
# systemctl status npm
```

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
echo url="https://www.duckdns.org/update?domains=nom-du-domaine&token=f30a8c59-b492-4323-...&ip=" | curl -k -o /root/duckdns/duck.log -K -
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

Tester le fonctionnement du script comme suit :

```bash
# cd /root/duckdns
# ./duck.sh
```

Un fichier duck.log sera créé et contiendra OK si tout s'est passé correctement.

La même opération peut par précaution être réalisée à l'intérieur du conteneur.

#### Certificat Let's Encrypt

Pour remplacer le numéro du port HTTPS 443 qu'utilise par défaut _Nginx Proxy Manager_, procéder ainsi à l'intérieur du conteneur :

```bash
# cd /etc/nginx/conf.d
# nano default.conf
```

Remplacer tous les 443 par le nouveau numéro de port.

Faire de même avec le fichier /app/templates/_listen.conf.

Redémarrer le conteneur et vérifier les ports d'écoute avec la Cde suivante :

```bash
# ss -tnalup
```

Installer le plugin certbot pour le fournisseur DuckDNS :

```bash
# pip install certbot_dns_duckdns
```

Créer enfin un certificat en utilisant le _DNS Challenge_ ce qui évite l'obligation d'être en écoute sur les ports HTTP 80 et HTTPS 443.

Penser à entrer un délai de 120 secondes afin d'éviter un éventuel problème de timeout lors de la création.

#### En-têtes des Proxy Hosts

Certains _Proxy Hosts_ peuvent nécessiter d'ajouter dans leur configuration des en-têtes HTTP.

Dans ce cas, ajouter le contenu suivant dans l'onget _Advanced_ de l'hôte en zone _Custom Nginx Configuration_ :

```markdown
location /
{
    proxy_pass http://IP-du-conteneur:8080;
    proxy_set_header Host sous-domaine.domaine.duckdns.org:7230;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header REMOTE-HOST $remote_addr;
    client_body_buffer_size 512k;
    proxy_read_timeout 86400s;
    client_max_body_size 0;
    proxy_buffers 256 16k;
    proxy_buffer_size 16k;
    add_header X-Cache $upstream_cache_status;
}
```

**Fin.**
