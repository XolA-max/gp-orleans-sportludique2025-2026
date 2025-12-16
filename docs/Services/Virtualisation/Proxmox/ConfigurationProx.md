# Proxmox
## Installation RAID 5 avec LVM sur le serveur

#### Commandes

```h
# Création d'un RAID 5 avec 4 disques
mdadm --create --verbose /dev/md0 --level=5 --raid-devices=4 /dev/sd[b-d]admin / admin

# Vérification de l'état du RAID
cat /proc/mdstat
mdadm --detail /dev/md0

# Initialiser le volume physique
pvcreate /dev/md0

# Créer un groupe de volumes
vgcreate vg_raid5 /dev/md0

# Créer un volume logique
lvcreate -L 5200G -n lv_proxmox vg_raid5
```
----------------------------------------------------------------------- 

### Configuration de Proxmox


```h
# Éditer la configuration réseau
nano /etc/network/interfaces

# Exemple de configuration VLAN-aware
auto vmbr0
iface vmbr0 inet static
    address X.X.X.X/24

# Redémarrer le réseau
systemctl restart networking
```

#### Gestion

L’administration s’effectue depuis l’interface web, disponible à l’adresse IP : 192.168.140.75

---