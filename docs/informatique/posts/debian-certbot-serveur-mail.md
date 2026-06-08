---
title: Debian - Certbot + Dynu + Postfix
summary: Création certificats Let's Encrypt pour Postfix.
authors: 
  - G.Leloup
date: 2026-06-08
categories: 
  - Debian
---

## Certbot et plugin Dynu

🧩 1. Installation de Certbot et du plugin DNS de `dynu.com` indispensable pour le challenge DNS‑01.

Commencer par installer les modules Python de nom pip et venv :

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

Le module venv sera utilisé pour créer l'environnement virtuel Python.  
Le module pip sera utilisé pour l'installation de Certbot et du plugin DNS Dynu.

Créer ensuite un dossier dédié à Certbot et installer l'environnement virtuel Python :

```bash
mkdir -p /home/user-x/certbot
python3 -m venv /home/user-x/certbot/
```

Puis, activer l’environnement virtuel Python :

```bash
source /home/user-x/certbot/bin/activate
```

Installer enfin Certbot et le plugin Dynu :

```bash
cd /home/user-x/certbot
pip install --upgrade certbot certbot-dns-dynu
pip show certbot
pip show certbot-dns-dynu

deactivate
```

Les fichiers sont installés dans /home/user-x/certbot/.

## Fichier credentials Dynu

🔐 2. Créer le fichier credentials qui contiendra l'API Key du compte Dynu.

<!-- more -->

Au préalable, récupérer le token API-KEY de Dynu ici :  
[https://www.dynu.com/en-US/ControlPanel/APICredentials](https://www.dynu.com/en-US/ControlPanel/APICredentials){ target="_blank" }

Créer ensuite les dossiers et fichiers suivants :

```bash
sudo mkdir -p /etc/letsencrypt
sudo mkdir -p /etc/letsencrypt/renewal-hooks/deploy
sudo nano /etc/letsencrypt/dynu.ini
```

et entrer dans le fichier dynu.ini le contenu suivant :

```markdown
dns_dynu_auth_token = Xe5eY4.....
```

Pour finir, sécuriser le fichier :

```bash
sudo chmod 600 /etc/letsencrypt/dynu.ini
```

## Certificats Let’s Encrypt

🚀 3. Générer les certificats Let’s Encrypt DNS‑01.

Cde exacte tenant compte du domaine :

```bash
sudo /home/user-x/certbot/bin/certbot certonly \
  --authenticator dns-dynu \
  --dns-dynu-credentials /etc/letsencrypt/dynu.ini \
  -d mail.geekcom82.freeddns.org
```

Certbot va :  
\- Créer automatiquement l’enregistrement TXT via Dynu  
\- Valider le domaine mail.geekcom82.freeddns.org  
\- Générer les certificats  
\- Les stocker dans /etc/letsencrypt/live/mail.geekcom82.freeddns.org/

!!! note "Nota"
    Attention l'enregistrement TXT ne pourra se faire que dans la limite de 4 enregistrements acceptés par Dynu pour un domaine gratuit.

En cas de problème voir les possibilité offertes par **CloudFlare**.

## Renouvellement automatique

🔄 4. Renouvellement automatique + rechargement de Postfix/Dovecot.

Certbot installe déjà un cron systemd pour le renouvellement.  
Il faut juste ajouter un hook pour recharger Postfix et Dovecot après le renouvellement.

Créer pour cela le fichier reload-mail.sh :

```bash
sudo nano /etc/letsencrypt/renewal-hooks/deploy/reload-mail.sh
```

et entrer le contenu suivant :

```markdown
#!/bin/bash
systemctl reload postfix
systemctl reload dovecot
```

Rendre le fichier exécutable :

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-mail.sh
```

## Intégration dans Postfix

📬 5. Intégrer les certificats dans Postfix.

Mettre dans /etc/postfix/main.cf :

```markdown
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.geekcom82.freeddns.org/fullchain.pem
smtpd_tls_key_file = /etc/letsencrypt/live/mail.geekcom82.freeddns.org/privkey.pem
smtpd_tls_CAfile = /etc/letsencrypt/live/mail.geekcom82.freeddns.org/chain.pem

smtpd_use_tls = yes
smtpd_tls_security_level = may
smtp_tls_security_level = may

smtpd_tls_auth_only = yes
```

!!! Warning "Important"
    La directive **smtpd_tls_auth_only = yes** impose que l’authentification SMTP (AUTH LOGIN/PLAIN) ne soit autorisée que si la connexion est chiffrée (TLS).  
    Sans cette directive, Postfix peut accepter une authentification en clair si un client se connecte en non‑TLS sur le port 587 ou 25.

Puis :

```bash
sudo systemctl restart postfix
```

## Integration dans Dovecot

📥 6. Intégrer les certificats dans Dovecot.

Mettre dans /etc/dovecot/conf.d/10-ssl.conf :

```markdown
ssl = yes
ssl_cert = </etc/letsencrypt/live/mail.geekcom82.freeddns.org/fullchain.pem
ssl_key = </etc/letsencrypt/live/mail.geekcom82.freeddns.org/privkey.pem
ssl_ca = </etc/letsencrypt/live/mail.geekcom82.freeddns.org/chain.pem
```

Puis :

```bash
sudo systemctl restart dovecot
```

**Fin.**
