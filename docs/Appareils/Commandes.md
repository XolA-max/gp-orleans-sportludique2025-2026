# Rappel des commandes

## Switch

#### 🔐 Configuration SSH et utilisateurs sur le switch cœur

```h
Coeur> enable
Coeur# conf t
Coeur(config)# enable secret "MotDePasse" [Pour mettre un MDP sur la commande enable]
Coeur(config)# ip domain name "NomDeDomaine.com"
Coeur(config)# ip ssh version 2 
Coeur(config)# crypto key generate rsa
 How many bits in the modulus [512]: 2048
Coeur(config)# username "NomDeL'Utilisateur" privilege 15 secret "MotDePasse" 
*Coeur(config)# line vty 0 15 
Coeur(config)# login local
Coeur(config)# transport input ssh 
Coeur(config)# exit
Coeur# exit
Coeur# write memory
``` 

### 🗂️ Configuration des VLAN :

#### Création d’un VLAN

```h
Switch(config)# vlan 10
Switch(config-vlan)# name Utilisateurs
Switch(config)# vlan 20
Switch(config-vlan)# name Serveurs
```

#### Suppression d’un VLAN

```h
Switch(config)# no vlan 20
```

#### Vérification des VLAN existants

```h
Switch# show vlan brief
```

### 🚦 Configuration des ports : Trunk et Access

#### Mode Trunk

```h
Switch(config)# interface GigabitEthernet1/0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20
Switch# show interfaces trunk
Switch# show running-config
```

#### Mode Access

```h
Switch(config)# interface GigabitEthernet1/0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch# show interfaces status
Switch# show running-config

```

#### 🖧 Stack

```h
Switch# show version              
Switch# show switch               
Switch(config)# switch <num> priority 15   
Switch# show switch stack-ports   
Switch# write memory                        
```

#### 🔗 LACP et EtherChannel


```h
Switch# show etherchannel summary             
Switch(config)# interface range GigabitEthernet1/0/1 - 2  
Switch(config-if-range)# channel-group 1 mode active       
Switch(config)# interface Port-channel 1       
Switch(config-if)# switchport mode trunk       
Switch(config-if)# switchport trunk allowed vlan 10,20  
Switch# show running-config                   
Switch# show interfaces status                
Switch# show etherchannel detail              
```

---

## Routeurs

### 🌐 Routage

#### Activation Routage

```h
Switch(config)# ip routing
```

#### Passerelle par défaut

```h
Switch(config)# ip route 0.0.0.0 0.0.0.0 <IP_Gateway>
```

#### Routes statiques

Ajouter des routes vers des réseaux spécifiques :  

```h
Switch(config)# ip route <Réseau1> <Masque1> <Next_Hop1>
Switch(config)# ip route <Réseau2> <Masque2> <Next_Hop2>
Switch(config)# ip route <Réseau3> <Masque3> <Next_Hop3>
```

#### Vérification

Vérifier les routes configurées :  

```h
Switch# show ip route
```

### 🧩 Encapsulation Dot1Q

### Configuration d’une sous-interface pour le routage inter-VLAN

Configurer une interface routeur pour transporter plusieurs VLANs via **802.1Q** :  

```h
Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
```

### 🛡️ ACL

#### ACL pour autoriser le VLAN Interco

Créer une ACL pour autoriser le trafic du VLAN Interco vers Internet :  

```h
Router(config)# access-list 100 permit ip 192.168.10.0 0.0.0.255 any
```

#### Appliquer l’ACL sur l’interface sortante

```h
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group 100 out
```

#### Vérification

```
Router# show access-lists
```

### 🔀 NAT/PAT

#### NAT pour traduire les adresses internes en IP publique

```
Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255
Router(config)# ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

#### Définir les interfaces NAT Inside / Outside

```h
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat inside
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat outside
```

#### Vérification

```h
Router# show ip nat translations
Router# show ip nat statistics
```

### 🛠️ HSRP (Hot Standby Router Protocol)

#### Configuration sur le premier routeur (Routeur FIBRE)

```h
RouterA(config)# interface GigabitEthernet0/1
RouterA(config-if)# ip address 192.168.1.2 255.255.255.0
RouterA(config-if)# standby 1 ip 192.168.1.1
RouterA(config-if)# standby 1 priority 110
RouterA(config-if)# standby 1 preempt
```

#### configuration sur le second routeur (Routeur ADSL)

```h
RouterB(config)# interface GigabitEthernet0/1
RouterB(config-if)# ip address 192.168.1.3 255.255.255.0
RouterB(config-if)# standby 1 ip 192.168.1.1
RouterB(config-if)# standby 1 priority 100
RouterB(config-if)# standby 1 preempt
```

### Installation RAID 5 avec LVM sur le serveur

#### Commandes

```h
# Création d'un RAID 5 avec 4 disques
mdadm --create --verbose /dev/md0 --level=5 --raid-devices=4 /dev/sd[b-d]

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

### Configuration de Proxmox

#### Configuration de Proxmox

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

## FireWall physique

### Accès à l’interface Web d’administration a l'installation

```h
https://10.0.0.254

admin / admin
```


### Définir les interfaces réseau

Identifier les VLANs utilisés dans le réseau interne (IN) avec leur **nom**, **interface associée (IN)**, **numéro de VLAN**, ainsi que **l’adresse de management** accompagnée de son **masque de sous-réseau**.  
Répéter la même opération pour les autres VLANs présents sur l’infrastructure.  

Désactiver l’interface **IN** principale et **laisser uniquement les VLANs actifs**.  

Enfin, identifier l’adresse de sortie (vers l’extérieur) en précisant son **nom**, son **adresse IP** et son **masque de sous-réseau**.


### Navigation

Après avoir configuré les ports LAN et WAN, la configuration se fait depuis l’interface web de l’équipement avec l'addresse LAN de Mana.

### Définir les réseaux de destination

Passer en **mode écriture**.

Aller dans :  
`Objets → Réseaux → Type : Réseau → Ajouter`

Renseigner le **nom des réseaux distants** ainsi que **leurs adresses IP** et **leurs masques**.

---

### Définir les passerelles

Aller dans :  
`Objets → Réseaux → Type : Machine → Ajouter`

Attribuer un **nom** et renseigner **l’adresse IP de la passerelle** correspondante.

---

### Ajouter les routes de retour

Aller dans :  
`Réseau → Routage → Ajouter`

Renseigner le **réseau de destination**, **l’interface de sortie** et la **passerelle** associée.



### Régle de filtarge

|Protocol |	Source |	Port |	Destination |	Port |	Gateway |
|---------|--------|---------|--------------|--------|----------|
|*        |*       |*        |*             |*       |*         |

Cela permet de n’avoir aucun problème au niveau des règles de filtrage.

### Desactiver le mode furtif

Aller dans :  
`Protection de sécurité → Protocoles → Protocoles IP → IP → Mode furtif`


---

## FireWall virtuel

OPNsense fonctionne sur une machine virtuelle Linux.

### Accès à l’interface Web d’administration

L’accès à l’interface web d’administration se fait grâce à l’adresse IP configurée lors de l’installation d’OPNsense.

### Définir les interfaces réseau

La configuration des interfaces réseau se fait en ligne de commande après l’installation.  
Il est important d’identifier correctement les adresses MAC et de les associer aux bonnes adresses IP, aussi bien pour les interfaces LAN (Management) que WAN.

---

### Navigation

Après avoir configuré les ports LAN, WAN et DMZ, la configuration s’effectue depuis l’interface web de l’équipement, en utilisant l’adresse IP LAN du réseau Management.  
Il faut également attribuer une adresse IP à l’interface DMZ, correspondant au sous-réseau de la DMZ.

---

### Définir les passerelles

Sur les interfaces DMZ et WAN, il est nécessaire de renseigner les passerelles (gateway) en leur attribuant un **nom** et une **adresse IP** correspondante.

---

### Règles de filtrage

Appliquer les règles suivantes sur les interfaces WAN et DMZ :

| Protocole | Source | Port source | Destination | Port destination | Passerelle |
|------------|---------|--------------|--------------|------------------|-------------|
| *          | *       | *            | *            | *                | *           |

Cette configuration permet de ne rencontrer aucun blocage au niveau du filtrage, le temps de valider le bon fonctionnement du réseau.
