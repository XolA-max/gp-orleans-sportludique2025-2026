# Configuration Proxmox

---

## Installation RAID 5 avec LVM

### Commandes de création RAID

!!! info
    ```bash
    # Création d'un RAID 5 avec 4 disques
    mdadm --create --verbose /dev/md0 --level=5 --raid-devices=4 /dev/sd[b-d]

    # Vérification de l'état du RAID
    cat /proc/mdstat
    mdadm --detail /dev/md0
    ```

### Configuration LVM

!!! info
    ```bash
    # Initialiser le volume physique
    pvcreate /dev/md0

    # Créer un groupe de volumes
    vgcreate vg_raid5 /dev/md0

    # Créer un volume logique
    lvcreate -L 5200G -n lv_proxmox vg_raid5
    ```

---

---

## Configuration Réseau (VLAN-aware)

Pour configurer les interfaces réseau :

!!! info
    `nano /etc/network/interfaces`

    ```bash
    auto vmbr0
    iface vmbr0 inet static
        address 192.168.140.75/24
        gateway 192.168.140.254
        bridge-ports eno1
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
    ```

    Appliquer :
    ```bash
    systemctl restart networking
    ```

---

---

## Administration

L’administration s’effectue depuis l’interface web :

> **https://192.168.140.75:8006**