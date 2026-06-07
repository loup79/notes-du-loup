---
title: VM KMV NixOS
summary: Installation du système Linux NixOS.
authors: 
  - G.Leloup
date: 2026-05-24
categories: 
  - NixOS
---

Voir par la suite Securix et Bureautix de l'ANSSI.

Liens utiles :  
[Linuxtricks.fr](https://www.linuxtricks.fr/wiki/nixos-installer-nixos-facilement-depuis-l-image-minimale){ target="_blank" }

[Le blog de Stéphane Robert](https://blog.stephane-robert.info/docs/securiser/os-immuable/nixos/){ target="_blank" }

[Manuel de NixOS](https://nixos.org/manual/nixos/stable/index.html#ch-installation){ target="_blank" }

[DamyFr](https://github.com/DamyrFr/nixos-laptops-config/blob/main/README.md){ target="_blank" }

[Paquets NixOS](https://github.com/NixOS/nixpkgs){ target="_blank" }

[Recherche de paquets](https://search.nixos.org/packages){ target="_blank" }

## Mémo KVM

NixOS sera une VM KVM installée sur le PC hôte U59 _(Debian 13)_.

Cdes à exécuter sur le PC hôte des VM.

### Installation

```bash
sudo apt install -y qemu-system-x86 libvirt-daemon-system libvirt-clients bridge-utils virt-manager
sudo systemctl enable --now libvirtd
```

Le paquet bridge-utils peut être remplacé par le paquet iproute2 plus récent.

Ajouter l'utilisateur user-x aux groupes libvirt et kvm :

```bash
sudo usermod -aG kvm,libvirt user-x
```

### - _Réseau pour les VM_

Le paquet network-manager doit être installé sur le PC hôte pour pouvoir utiliser la Cde nmcli.

Créer un bridge br0, ceci permettra de mettre les VM directement sur le LAN :

```bash
sudo nmcli connection add type bridge ifname br0
```

Ajouter une interface physique _(par exemple enp2s0)_ au bridge et activer celui-ci :

```bash
sudo nmcli connection add type bridge-slave ifname enp2s0 master br0
sudo nmcli connection down enp2s0
sudo nmcli connection up bridge-br0
```

Vérifier enfin le statut du bridge :

```bash
ip addr show br0
```

et redémarrer le démon libvirtd :

```bash
sudo systemctl restart libvirtd
```

### - _Démarrage auto des VM_

```bash
ls -l /etc/libvirt/qemu/autostart/
```

Les fichiers listés doivent être en -rw-r--r-- et appartenir à root:root.

<!-- more -->

### - _Exportation d'une VM_

Le but est de sauvegarder la VM.

```bash
sudo ls -l /var/lib/libvirt/images/nom-vm/
```

Le contenu du dossier nom-vm doit montrer le disque QCOW2 de la VM.

Lecture du fichier xml associé :

```bash
sudo virsh dumpxml nom-vm
```

Cdes pour copier le disque QCOW2 et le fichier xml sur un autre disque :

```bash
sudo cp /var/lib/libvirt/images/nom-vm/nom-vm.qcow2 /mnt/ssd-interne/QCOW2-U59/
virsh dumpxml nom-vm > /mnt/ssd-interne/QCOW2-U59/nom-vm.xml
```

### - _Fichiers importants de KVM_

| Localisation | |
| ------------- | ------------------------- |
| /etc/libvirt/ | Fichiers de configuration |
| /etc/libvirt/qemu | Fichiers xml des VM |
| /var/lib/libvirt/images/ | Disques QCOW2 des VM |
| /run/libvirt/libvirt-sock | Socket de communication |
| /var/log/libvirt/qemu/ | Logs des VM |

## NixOS

### Création et connexion SSH

Récupérer l'ISO minimale :

```bash
mkdir -p /home/user-x/test-nixos/images && cd /home/user-x/test-nixos/images
wget https://channels.nixos.org/nixos-26.05/latest-nixos-minimal-x86_64-linux.iso
```

Créer enfin la VM nixos-loup avec 8 Go de RAM comme suit :

```bash
sudo virt-install \
  --connect qemu:///system \
  --name nixos-loup \
  --ram 8192 \
  --vcpus 2 \
  --disk size=20,format=qcow2,pool=default,bus=virtio \
  --cdrom /home/user-x/test-nixos/images/latest-nixos-minimal-x86_64-linux.iso \
  --os-variant nixos-unstable \
  --network bridge=br0,model=virtio \
  --graphics vnc,listen=127.0.0.1 \
  --boot loader=/usr/share/OVMF/OVMF_CODE_4M.fd,loader.readonly=yes,loader.type=pflash,loader.secure=no \
  --noautoconsole
```

La VM est normalement créée dans le dossier /var/lib/libvirt/images/ sous le nom de nixos-loup.qcow2.

Vérifier si la VM nixos-loup apparaît bien dans la liste des VM existantes :

```bash
sudo virsh list --all
```

Ensuite ouvir la console de la VM nixos-loup depuis l'outil graphique Virtual Machine Manager.

Passer le clavier en azerty :

```bash
sudo loadkeys.fr # Attention à la lettre a
```

Affecter un MDP à root :

```bash
sudo passwd root
```

Récupérer l'adresse IP qui a été attribuée à la VM :

```bash
ip a
```

et vérifier l'activation par défaut d'un serveur SSH :

```bash
sudo systemctl status sshd
```

> Connexion SSH sur la VM nixos-loup :  
> Des clés SSH publiques et privées peuvent être générées avec la Cde ssh-keygen du paquet openssh-client.  
> Le PC hôte ou le PC distant doivent montrer la paire de clés ssh dans leur dossier caché .ssh.

Tester ensuite une connexion SSH depuis le PC hôte ou un PC distant situé sur le même réseau local :

```bash
sshpass -p 'mdp-habituel' ssh -o StrictHostKeyChecking=no root@ip-vm-nixos-loup
```

La connexion doit s'établir.

### Configuration

#### - _Préparation (3 fichiers)_

Sur le PC hôte, créer les 3 fichiers suivants dans le dossier /home/user-x/test-nixos/config :

\- Fichier flake.nix

```markdown hl_lines="5 13 17 18"
{
  description = "NixOS — VM KVM de base";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-26.05";
    disko = {
      url = "github:nix-community/disko";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, disko }: {
    nixosConfigurations.nixos-loup = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        disko.nixosModules.disko
        ./disko-config.nix
        ./configuration.nix
      ];
    };
  };
}
```

nixos-26.05 = Channel ou dépôt utilisé, exemple : `https://channels.nixos.org/nixos-26.05`  
nixos-loup = Nom d'hôte de la VM

\- Fichier disco-config.nix

```markdown
{ lib, ... }:
{
  disko.devices = {
    disk = {
      main = {
        type = "disk";
        device = "/dev/vda";
        content = {
          type = "gpt";
          partitions = {
            ESP = {
              size = "512M";
              type = "EF00";
              content = {
                type = "filesystem";
                format = "vfat";
                mountpoint = "/boot";
                mountOptions = [ "umask=0077" ];
              };
            };
            root = {
              size = "100%";
              content = {
                type = "filesystem";
                format = "ext4";
                mountpoint = "/";
              };
            };
          };
        };
      };
    };
  };
}
```

\- Fichier configuration.nix

```markdown hl_lines="39 42"
{ config, pkgs, modulesPath, lib, ... }:
{
  # Profil QEMU : charge les modules virtio dans l'initrd
  imports = [ (modulesPath + "/profiles/qemu-guest.nix") ];

  # Bootloader UEFI
  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;

  # Modules noyau pour la virtualisation
  boot.initrd.availableKernelModules =
    [ "virtio_pci" "virtio_blk" "virtio_scsi" "ahci" "sd_mod" ];
  boot.kernelModules = [ "virtio_net" ];

  # Nom d'hôte sur le réseau
  networking.hostName = "nixos-loup";

  # Locale et timezone
  time.timeZone = "Europe/Paris";
  i18n.defaultLocale = "fr_FR.UTF-8";
  console.keyMap = "fr";

  # SSH sécurisé
  services.openssh = {
    enable = true;
    settings = {
      PermitRootLogin = "prohibit-password";
      PasswordAuthentication = false;
    };
  };

  # PAM
  security.pam.services.xrdp-sesman.allowNullPassword = lib.mkForce false;
  security.pam.sshAgentAuth.enable = false;

  # Utilisateur user-x comme administrateur avec accès SSH par clé
  users.users.user-x = {
    isNormalUser = true;
    hashedPassword = "$6$j94Bzh4pkDsc4eE9...";
    extraGroups = [ "wheel" ];
    openssh.authorizedKeys.keys = [
      "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKU1GMq... loup@loup-u59"
      "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFhz3VY... ed25519 256-20260604"
    ];
  };

  # Sudo sans mot de passe (VM de test uniquement)
  security.sudo.wheelNeedsPassword = false;

  # Règles de pare-feu
  networking.firewall = {
  enable = true;
  allowedTCPPorts = [ 22 3389 ];
  };

  # Paquets de base
  environment.systemPackages = with pkgs; [
    # Paquets de base
    vim git htop curl tmux nano

    # paquets du loup
    xfce4-whiskermenu-plugin
    xfce4-pulseaudio-plugin
    xfce4-screenshooter
    xfce4-taskmanager 
    xfce4-terminal

    firefox
    
    thunar
    thunar-volman
    thunar-archive-plugin
  ];

  # Activer flakes et nix-command
  nix.settings.experimental-features = [ "nix-command" "flakes" ];

  # Garbage collection automatique
  nix.gc = {
    automatic = true;
    dates = "weekly";
    options = "--delete-older-than 30d";
  };

  system.stateVersion = "26.05";

  # Installation de xorg, lightdm, xfce de base, networkmanager, polkit et audio pipewire
  services.xserver.enable = true;
  services.xserver.displayManager.lightdm.enable = true;
  services.xserver.desktopManager.xfce.enable = true;

  # Réseau DHCP simple
  networking.networkmanager.enable = true;
  
  security.polkit.enable = true;
  
  services.pipewire = {
  enable = true;
  alsa.enable = true;
  pulse.enable = true;
  };
  
  ## Services GVFS sous xfce (montage USB, réseau, etc.)
  services.gvfs.enable = true;
  services.tumbler.enable = true;

  ## Services systemd XRDP pour xfce
  services.xrdp.enable = true;
  services.xrdp.defaultWindowManager = "xfce4-session";
}
```

#### - _Remarques_

Entrer ceci dans la console nixos-loup afin que XRDP fonctionne avec XFCE :

```bash
touch /home/loup/.xsession
echo "xfce4-session" > /home/loup/.xsession
sudo chown loup:users /home/loup/.xsession
ls -la /home/loup/
```

Le hash du MDP de l'utilisateur user-x a été obtenu avec la Cde :

```bash
mkpasswd -m sha-512
```

La clé SSH publique déclarée est celle se trouvant dans le dossier caché .ssh du PC hôte.

Il est possible de déclarer ci-dessus plusieurs clés publiques SSH selon le nombre de clients distants.

#### - _Copie de la configuration_

Depuis le client SSH du PC hôte, copier les 3 fichiers de configuration sur la VM nixos-loup :

```bash
sshpass -p 'mdp-habituel' scp -o StrictHostKeyChecking=no \
/home/user-x/test-nixos/config/*.nix root@ip-vm-nixos-loup:/tmp/nixos-config/
```

Vérifier ensuite le résultat dans la console de la VM nixos-loup :

```bash
sudo ls -la /tmp/nixos-config
```

#### - _Initialiser le dépot Git_

Dans la VM, les flakes exigent un dépôt Git temporaire pour déterminer quels fichiers inclure :

```bash
su root
cd /tmp/nixos-config
git init
git config user.email "user-x@nixos-loup"
git config user.name "user-x"
git add .
git commit -m "initial nixos config"
```

On peut voir dans la dernière Cde l'inclusion des 3 fichiers de configuration.

#### - Verrouiller les dépendances

```bash
nix --extra-experimental-features "nix-command flakes" flake lock
git add flake.lock
git commit -m "add flake.lock"
```

Cette commande télécharge les index de nixpkgs et disko, puis génère flake.lock avec les commits exacts.  
C'est ce fichier qui garantit la reproductibilité.

### Préparer le disque

```bash
nix --extra-experimental-features "nix-command flakes" run github:nix-community/disko -- --mode disko /tmp/nixos-config/disko-config.nix
```

Vérification du résultat :

```bash
lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT /dev/vda
```

Retour normal :

\# NAME   FSTYPE  SIZE MOUNTPOINT  
\# vda             20G  
\# ├─vda1 vfat    512M /mnt/boot  
\# └─vda2 ext4   19,5G /mnt

Voir par la suite l'ajout de nix.settings.experimental-features = [ "nix-command" "flakes" ]; dans le fichier configuration.nix.

### Installation de l'OS NixOS

Le dossier /mnt contient de base les dossiers boot et lost+found.

Copier la configuration dans le futur système

```bash
mkdir -p /mnt/etc/nixos
cp -r /tmp/nixos-config/* /mnt/etc/nixos/
```

et lancer l'installation depuis le client SSH du PC hôte :

```bash
nixos-install --flake /mnt/etc/nixos#nixos-loup --no-root-passwd
```

L'installation télécharge les paquets depuis le cache binaire (environ 2,5 Go), construit la configuration finale et installe le bootloader. Le message "installation finished!" confirme le succès.

--no-root-passwd est utilisé ici car notre configuration n'autorise que l'accès SSH par clé.  
Il n'y a pas de mot de passe root à définir.

Ensuite sur le PC hôte de la VM, ejecter le CD-ROM et rebooter :

```bash
sudo virsh change-media nixos-loup sda --eject
sudo virsh reboot nixos-loup
```

Enfin, se connecter en SSH depuis le PC hôte :

```bash
ssh user-x@ip-vm-nixos-loup
```

La connexion doit s'établir.

Si problème de login au reboot, entrer sur le PC hôte la Cde ci-dessous et tester une nouvelle connexion :

```bash
ssh-keygen -f '/home/user-x/.ssh/known_hosts' -R 'ip-vm-nixos-loup'
```

## Contrôles post installation

Vérifier la version de Nix OS :

```bash
nixos-version
```

Retour :

```markdown
26.05.20260531.b51242d (Yarara)
```

Vérifier le noyau et le hostname :

```bash
uname -r && hostname
```

Retour :

```markdown
6.18.33
nixos-loup
```

Vérifier le partitionnement :

```bash
lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT /dev/vda
```

Retour normal :

```markdown
NAME   FSTYPE  SIZE MOUNTPOINT
vda             20G 
├─vda1 vfat    512M /boot
└─vda2 ext4   19,5G /
```

Vérifier l'espace disque :

```bash
df -h / /boot
```

Retour :

```markdown
Sys. de fichiers Taille Utilisé Dispo Uti% Monté sur
/dev/vda2           20G    2,7G   16G  15% /
/dev/vda1          511M     41M  471M   8% /boot
```

Vérifier SSH et la sécurité :

```bash
systemctl is-active sshd

grep -E "^(PermitRootLogin|PasswordAuthentication)" /etc/ssh/sshd_config
```

Retours :

```markdown
active

PasswordAuthentication no
PermitRootLogin prohibit-password
```

Vérifier la configuration déclarative :

```bash
ls /nix/store/ | grep nixos-system-nixos
cd /nix/store/q9cavwpcv8q1z6z0kphm2acdqxrd1s61-nixos-system-nixos-loup-26.05.20260531.b51242d

readlink /run/current-system
```

Retours :

```markdown
q9cavwpcv8q1z6z0kphm2acdqxrd1s61-nixos-system-nixos-loup-26.05.20260531.b51242d
```

Ce lien symbolique pointe vers la génération active dans le store Nix.  
Chaque rebuild créera une nouvelle génération.

Vérifier la locale :

```bash
echo $LANG
```

Retour normal :

```markdown
fr_FR.UTF-8
```

## Modification déclarative

Si l'on modifie par exemple le fichier configuration.nix il est nécessaire de reconstruire NixOS comme suit :

```bash
cd /etc/nixos
sudo nixos-rebuild switch --flake .#nixos-loup
```

Cela créé une nouvelle génération, celles-ci peuvent être listées comme suit :

```bash
sudo nix-env --list-generations -p /nix/var/nix/profiles/system
```

Exemple de retour :

```markdown
4   2026-06-04 09:57:22   
5   2026-06-04 11:06:13   (current)
```

Pour supprimer une génération :

```bash
sudo nix-env --delete-generations 4 --profile /nix/var/nix/profiles/system
```

Pour supprimer toutes les générations sauf la dernière :

```bash
sudo nix-collect-garbage -d
```

> Il n'est pas nécessaire de reconstruire NixOS suite à une suppression de génération.

Pour revenir sur la génération précédente _(à vérifier)_ :

```bash
sudo nixos-rebuild switch --rollback
```

Le rollback réactive l'ancienne génération à chaud, sans reboot.

Modifier le fichier configuration.nix afin d'éviter de retrouver le même problème.

**Fin.**
