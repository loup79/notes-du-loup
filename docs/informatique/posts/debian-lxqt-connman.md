---
title: Debian - LXQT Connman
description: Ajout d'une route permanente sous LXQT.
authors: 
  - G.Leloup
date: 2026-05-24
categories: 
  - Debian
---

Pour ajouter une route permanente alors que Connman gère le réseau, procéder ainsi :

```bash
sudo nano /etc/systemd/system/static-route.service
```

Entrer le contenu suivant :

```markdown
[Unit]
Description=Route statique vers 192.168.65.0/24
After=network-online.target connman.service
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ip route replace 192.168.65.0/24 via 192.168.3.13 dev enp0s3
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Exécuter les Cdes suivantes pour traiter le service :

```bash
sudo systemctl daemon-reload
sudo systemctl enable static-route.service
sudo systemctl start static-route.service
sudo systemctl status static-route.service
```

Vérifier le résultat en listant la table de routage :

```bash
ip r
```

**Fin.**
