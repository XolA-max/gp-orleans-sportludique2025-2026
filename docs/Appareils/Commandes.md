# Rappel des commandes 
## Équipements utilisés

- **Switch cœur :** Cisco 3750 L3
- **Routeurs :** Cisco 1921 
- **Serveurs :** [À compléter]  
## Informations utile
- **Les addresses Ip id de VLAN et numéro d'interfaces sont fictif  :** 

## Commandes
## Switch :
### 🔐 Configuration SSH et utilisateurs sur le switch cœur

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

## 🗂️ Configuration des VLAN :

## Création d’un VLAN
```h
Switch(config)# vlan 10
Switch(config-vlan)# name Utilisateurs
Switch(config)# vlan 20
Switch(config-vlan)# name Serveurs
```

### Suppression d’un VLAN 
```h
Switch(config)# no vlan 20
```
### Vérification des VLAN existants
```h
Switch# show vlan brief
```

## 🚦 Configuration des ports : Trunk et Access :

### Mode Trunk
```h
Switch(config)# interface GigabitEthernet1/0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20
Switch# show interfaces trunk
Switch# show running-config
```
### Mode Access
```h
Switch(config)# interface GigabitEthernet1/0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch# show interfaces status
Switch# show running-config

```
## 🖧 Stack :
```h
Switch# show version              
Switch# show switch               
Switch(config)# switch <num> priority 15   
Switch# show switch stack-ports   
Switch# write memory                        
```
## 🔗 LACP / EtherChannel :
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

# Routeurs
## 🌐 Routage :
### Activation Routage 
```h
Switch(config)# ip routing
```
### Passerelle par défaut 


```h
Switch(config)# ip route 0.0.0.0 0.0.0.0 <IP_Gateway>
```
### Routes statiques
Ajouter des routes vers des réseaux spécifiques :  
```h
Switch(config)# ip route <Réseau1> <Masque1> <Next_Hop1>
Switch(config)# ip route <Réseau2> <Masque2> <Next_Hop2>
Switch(config)# ip route <Réseau3> <Masque3> <Next_Hop3>
```
### Vérification
Vérifier les routes configurées :  
```h
Switch# show ip route
```

## 🧩 Encapsulation Dot1Q

## Configuration d’une sous-interface pour le routage inter-VLAN
Configurer une interface routeur pour transporter plusieurs VLANs via **802.1Q** :  

```h
Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
```
## 🛡️ ACL :

### ACL pour autoriser le VLAN Interco
Créer une ACL pour autoriser le trafic du VLAN Interco vers Internet :  
```h
Router(config)# access-list 100 permit ip 192.168.10.0 0.0.0.255 any
```
### Appliquer l’ACL sur l’interface sortante
```h
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group 100 out
```
### Vérification
```
Router# show access-lists
```
## 🔀 NAT/PAT :
### NAT pour traduire les adresses internes en IP publique
```
Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255
Router(config)# ip nat inside source list 1 interface GigabitEthernet0/0 overload
```
### Définir les interfaces NAT Inside / Outside
```h
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat inside
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat outside
```
### Vérification
```h
Router# show ip nat translations
Router# show ip nat statistics
```

## 🛠️ HSRP (Hot Standby Router Protocol)

### Configuration sur le premier routeur (Routeur FIBRE)

```h
RouterA(config)# interface GigabitEthernet0/1
RouterA(config-if)# ip address 192.168.1.2 255.255.255.0
RouterA(config-if)# standby 1 ip 192.168.1.1
RouterA(config-if)# standby 1 priority 110
RouterA(config-if)# standby 1 preempt
```

### configuration sur le second routeur (Routeur ADSL)
```h
RouterB(config)# interface GigabitEthernet0/1
RouterB(config-if)# ip address 192.168.1.3 255.255.255.0
RouterB(config-if)# standby 1 ip 192.168.1.1
RouterB(config-if)# standby 1 priority 100
RouterB(config-if)# standby 1 preempt
```

## Installation RAID 5 avec LVM sur le serveur

### Commandes
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

## Configuration de Proxmox

### Configuration de Proxmox
```h
# Éditer la configuration réseau
nano /etc/network/interfaces

# Exemple de configuration VLAN-aware
auto vmbr0
iface vmbr0 inet static
    address 192.168.140.75/24

# Redémarrer le réseau
systemctl restart networking
```
### Gestion

L’administration s’effectue depuis l’interface web, disponible à l’adresse IP : 192.168.140.75





