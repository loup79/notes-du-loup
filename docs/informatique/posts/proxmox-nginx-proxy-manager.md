---
title: "Proxmox - Nginx Proxy Manager"
summary: Notes diverses sur Proxmox.
authors: 
  - G.Leloup
date: 2024-02-04
categories: 
  - Proxmox
---

## Proxmox 9.x

Mise à jour le : 09/01/2026

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
[Documentation](https://nginxproxymanager.com/guide/){ target="_blank" }

#### Création du conteneur

Le conteneur Proxmox 9.x contenant l'application _Nginx Proxy Manager_ servira de _reverse proxy_ pour l'ensemble des VM/CTN de Proxmox.

Depuis l'interface Web de Proxmox :  
&rarr; Stockage local &rarr; Modèles de conteneurs  
&rarr; Bouton _Modèles_

Télécharger le modèle _debian-13-standard_.

Ensuite cliquer sur le bouton bleu _Créer un conteneur_ :  
-- Onglet Général  
Nom d'hôte : Ex _npmloup_  
Mot de passe : Entrer le MDP pour _root_  
Confirmer le mot de passe

-- Onglet Modèle  
Choisir le modèle _debian-13-standard_

-- Onglet Disques  
Stockage : local-lvm  
Taille du disque : 8 Go

-- Onglet Processeur  
Coeurs : 2 si possible

-- Onglet Mémoire  
Mémoire : 1024  
Espace d'échange : 512

-- Onglet Réseau  
Pont : vmbr0  
IPv4 : DHCP  
IPv6 : DHCP

-- Onglet DNS  
DNS : utiliser les valeurs de l'hôte

-- Onglet Confirmation  
Vérifier la configuration et cliquer sur le bouton _Terminer_.

Le conteneur est crée et apparaît dans le panneau de gauche.

#### Installation de docker

Sélectionner le conteneur _npmloup_ et démarrer celui-ci.

&rarr; Bouton _Console_ et se connecter en tant que _root_.

Mettre à jour Debian et installer les paquets suivants :

```bash
apt update
apt upgrade
apt install -y curl gnupg
```

Télécharger ensuite la clé gpg pour Docker :

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg
```

Puis déclarer le dépôt qui sera utilisé pour son installation :

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
$(. /etc/os-release && echo $VERSION_CODENAME) stable" \
> /etc/apt/sources.list.d/docker.list
```

Installer Docker :

```bash
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

et vérifier son son numéro de version :

```bash
docker --version
```

#### Installation de NPM

Au préalable, relever l'ID du conteneur _npmloup_ _(Ex : 109)_ et depuis le shell du noeud Proxmox :

```bash
cd /etc/pve/lxc/
nano 109.conf
```

Ajouter les 3 lignes suivantes en fin de fichier :

```markdown
lxc.apparmor.profile: unconfined
lxc.cap.drop:
lxc.cgroup2.devices.allow: a
```

Puis redémarrer le conteneur pour la suite et retourner dans celui-ci.

Créer ensuite le dossier suivant :

```bash
mkdir -p /opt/nginx-proxy-manager
cd /opt/nginx-proxy-manager
```

et créer un fichier _docker-compose.yml_ :

```bash
cat > docker-compose.yml << 'EOF'
version: "3"
services:
  app:
    image: jc21/nginx-proxy-manager:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "81:81"
      - "443:443"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
EOF
```

Il peut être nécessaire selon la configuration future de modifier le port d'écoute 443 côté externe.  
Dans ce cas, remplacer - "443:443" par Ex : - "78xx:443".

Lancer enfin l'installation de NPM :

```bash
docker compose up -d
```

L'interface WEB de _Nginx Proxy Manager_ devient accessible depuis l'URL :  
`http://ip-de-npmloup:81`

Entrer une adresse mail et un MDP pour ouvrir l'interface WEB de NPM.

#### Mise à jour de NPM

Pour effectuer les mises à jour, procéder ainsi :

```bash
cd /opt/nginx-proxy-manager
docker compose pull
docker compose up -ds
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

Ex : nom-du-domaine = wxyz pour wxyz.duckdns.org.

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

Créer un certificat en utilisant le _DNS Challenge_ ce qui évite l'obligation d'être en écoute sur les ports HTTP 80 et HTTPS 443.

Le token de duckdns.org sera demandé lors de la procédure.

Penser à entrer un délai de 120 secondes afin d'éviter un éventuel problème de timeout lors de la création.

#### En-têtes des Proxy Hosts

Certains _Proxy Hosts_ peuvent nécessiter d'ajouter dans leur configuration des en-têtes HTTP.

Dans ce cas, ajouter le contenu suivant dans l'onget _Advanced_ de l'hôte en zone _Configuration_ :

```markdown
location /
{
    proxy_pass http://IP-du-conteneur:8080;
    proxy_set_header Host sous-domaine.domaine.duckdns.org:7230;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
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
