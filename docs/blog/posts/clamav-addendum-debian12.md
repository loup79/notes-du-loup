---
title: "ClamAV - Addendum/Deb12"
summary: Pb avec le paquet clamav-unofficial-sigs, etc ...
authors: 
  - G.Leloup
date: 2025-03-28
categories: 
  - 12 Serveur de courrier
---

<figure markdown>
  ![Logo - ClamAV](../images/2025/03/clamav.webp)
</figure>

## Bdd de SecuriteInfo

### Paquet clamav-unofficial-sigs

Des courriers pour le compte _clamav@srvdmz_, reçus sur l'alias _postmaster@loupvirtuel.fr_, peuvent signaler les 2 incidents suivants :

\- curl: (6) Could not resolve host: clamav.**securiteinfo**...  
\- Clamscan ... **MalwarePatrol** mbl.ndb ... BAD - SKIP...

Ceux-ci traduisent un échec de MAJ de bases de données de signatures déclarées dans le fichier /usr/share/**clamav-unofficial-sigs**/conf.d/00-clamav-unofficial-sigs.conf.

Le fichier cron déclenchant les mises à jour se situe dans le dossier /etc/cron.d/.

Le paquet clamav-unofficial-sigs fourni par Debian semble être à l'origine de ces incidents.

Pour régler le problème, procédez ainsi :

```bash
[srvdmz@srvdmz:~$] cd /usr/share/clamav-unofficial-sigs/
[srvdmz@srvdmz:~$] cd conf.d
[srvdmz@srvdmz:~$] sudo nano 00-clamav-unofficial-sigs.conf 
```

Commentez les lignes dédiées aux Bdd **SecuriteInfo** :

```markdown
#si_dbs="
#   honeynet.hdb
#   securiteinfo.hdb
#   securiteinfobat.hdb
#   securiteinfodos.hdb
#   securiteinfoelf.hdb
#   securiteinfohtml.hdb
#   securiteinfooffice.hdb
#   securiteinfopdf.hdb
#   securiteinfosh.hdb
#"

#si_update_hours="4"
```

ainsi que celles dédiées à la Bdd **MalwarePatrol** :

<!-- more -->

```markdown
#mbl_dbs="
#   mbl.ndb
#"

#mbl_update_hours="6"
```

Puis, redémarrez les services suivants :

```bash
[srvdmz@srvdmz:~$] sudo systemctl restart clamav-daemon
[srvdmz@srvdmz:~$] sudo systemctl restart clamav-freshclam
```

!!! note "Nota"
    Les Bdd **SaneSecurity** déclarées dans 00-clamav-unofficial-sigs.conf continueront d'être mises à jour dans le dossier /var/cache/clamav-unofficial-sigs/ss-dbs/.

### Récupération Bdd SecuriteInfo

Il est possible de s'inscrire gratuitement sur le site [SecuriteInfo.com](https://www.securiteinfo.com){ target="_blank" } qui dans son offre Basic **malwares de 2012 jusqu'au mois dernier** permet de télécharger les signatures additionnelles pour ClamAV.

Utilisez le lien _Booster l'antivirus ClamaV_ présent sur la page d'accueil du site pour accéder à l'inscription.

Une fois fait, téléchargez comme proposé la liste des signatures et ajoutez celle-ci en fin du fichier /etc/clamav/freshclam.conf :

```markdown
# Bases de données de securiteinfo.com
DatabaseCustomURL https://www.securiteinfo.com/get/signatures/137ab3b.../securiteinfo.hdb

DatabaseCustomURL https://www.securiteinfo.com/get/signatures/137ab3d.../securiteinfo.ign2

Etc...
```

Redémarrez ensuite les services ci-dessous :

```bash
[srvdmz@srvdmz:~$] sudo systemctl restart clamav-daemon
[srvdmz@srvdmz:~$] sudo systemctl restart clamav-freshclam
```

Attendez quelque temps, MAJ effectuée 2 fois par jour, puis vérifiez le résultat :

```bash
[srvdmz@srvdmz:~$] sudo systemctl status clamav-freshclam
```

Retour :

```markdown hl_lines="3 15"
● clamav-freshclam.service - ClamAV virus database upd...
     Loaded: loaded (/lib/systemd/system/clamav-fresh...
     Active: active (running) since Mon 2024-10-... ago
       Docs: man:freshclam(1)
             man:freshclam.conf(5)
             https://docs.clamav.net/
   Main PID: 33880 (freshclam)
      Tasks: 1 (limit: 5824)
     Memory: 7.1M
        CPU: 700ms
     CGroup: /system.slice/clamav-freshclam.service
             └─33880 /usr/bin/freshclam -d --foregrou...

... freshclam... -> bytecode.cvd database is up-to-date
... freshclam... -> securiteinfo.hdb is up-to-date
... freshclam... -> securiteinfo.ign2 is up-to-date
... freshclam... -> javascript.ndb is up-to-date
... freshclam... -> spam_marketing.ndb is up-to-date
... freshclam... -> securiteinfohtml.hdb is up-to-date
... freshclam... -> securiteinfoascii.hdb is up-to-date
... freshclam... -> securiteinfoandroid.hdb is up-to-date
... freshclam... -> securiteinfoold.hdb is up-to-date
... freshclam... -> securiteinfopdf.hdb is up-to-date
```

Vous pouvez observer la MAJ des Bdd **SecuriteInfo**.

<center>---------- Fin ----------</center>
