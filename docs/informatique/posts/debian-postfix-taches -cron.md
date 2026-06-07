---
title: Debian - Postfix et tâches cron
description: Envoi d'un e-mail avec Postfix depuis une tâche cron.
authors: 
  - G.Leloup
date: 2026-05-24
categories: 
  - Debian
---

## Situation de départ

Plusieurs tâches cron actives sur le PC hote5 _(s51)_

Le but est que les notifications par e-mail issues des tâches cron arrivent bien à destination.

## Postfix

### Configuration

Editer le fichier de configuration de Postfix :

```bash
sudo nano /etc/postfix/main.cf
```

Puis modifier ou ajouter cette ligne afin d'éviter des erreurs avec le protocole IPv6 :

```markdown
inet_protocols = ipv4 
```

Gérer ensuite le paramètre relayhost pour le relais SMTP :

```markdown
relayhost = [exemple.relais.net]:587
```

Ainsi que les paramètres SASL et TLS pour le relais SMTP :

```markdown
# Gestion protocole SASL partie client SMTP de Postfix
smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtpd_client_restrictions = permit_sasl_authenticated,reject_unknown_client_hostname

smtp_use_tls = yes
smtp_tls_security_level = encrypt
smtp_tls_note_starttls_offer = yes
```

<!-- more -->

Créer le fichier /etc/postfix/sasl/sasl_passwd qui doit contenir :

```markdown
[exemple.relais.net]:587 userx@mail.relais.smtp.yz:mot-de-passe
```

Chiffrer ensuite son contenu :

```bash
sudo postmap hash:sasl_passwd
```

Un fichier de nom sasl_passwd.db est alors créé.

Protéger les permissions de sasl_passwd :

```bash
sudo chmod 600 sasl_passwd
```

Tous les fichiers et dossiers de /etc/postfix sont en root:root.

Redémarrer Postfix pour la prise en compte des modifications :

```bash
sudo systemctl restart postfix
```

### Réécriture d'adresses e-mail

Utilisation de la réécriture d'adresses e-mail locales vers des adresses e-mail externes.

Editer pour cela le fichier de configuration de Postfix :

```bash
sudo nano /etc/postfix/main.cf
```

et ajouter ces lignes en fin de fichier :

```markdown
## Gestion de l'expéditeur
smtp_generic_maps = hash:/etc/postfix/generic
sender_canonical_maps = hash:/etc/postfix/sender_canonical
```

Créer ensuite le premier fichier de réécriture de nom generic :

```bash
sudo nano /etc/postfix/generic
```

et y entrer le contenu suivant :

```markdown
root@hote5    userx@mail.yz
root          userx@mail.yz
```

Puis compiler celui-ci :

```bash
sudo postmap /etc/postfix/generic
```

Cela génère un fichier de nom generic.db.

Ce fichier sert à réécrire l’adresse juste avant l’envoi vers l’extérieur, c'est l'adresse que verront les destinataires.

Créer le second fichier de réécriture de nom sender_canonical :

```bash
sudo nano /etc/postfix/sender_canonical
```

et y entrer le contenu suivant :

```markdown
@hote5.home userx@mail.relais.smtp.yz
```

Puis compiler celui-ci :

```bash
sudo postmap /etc/postfix/sender_canonical
```

Cela génère un fichier de nom sender_canonical.db.

Ce fichier sert à présenter au relais SMTP une adresse qu’il accepte et c’est l’adresse interne utilisée par Postfix pour l’envoi.

Terminer en redémarrant Postfix.

```bash
sudo systemctl restart postfix
```

## Test d'envoi d'un e-mail

Envoyer un e-mail vers l'utilisateur root :

```bash
echo "test local" | mail -s "test" root
```

Ainsi que vers cette adresse e-mail externe `usery@gmail.com` :

```bash
echo "test externe" | mail -s "test relay" `usery@gmail.com`
```

Puis regarder dans les logs :

```bash
sudo journalctl -u postfix -n 50
```

L’expéditeur de l'e-mail devrait maintenant être `userx@mail.yz`.

## Fichier crontab

```bash
sudo -u root crontab -e
```

Vérifier l'existence de ces lignes sinon les ajouter :

```markdown
MAILFROM= userx@mail.relais.smtp.yz
MAILTO= usery@gmail.com
```

## Purge de la file d'attente

Au préalable, afficher le contenu de la file d'attente :

```bash
sudo postqueue -p
```

Purger ensuite celle-ci :

```bash
sudo postsuper -d ALL
```

**Fin.**
