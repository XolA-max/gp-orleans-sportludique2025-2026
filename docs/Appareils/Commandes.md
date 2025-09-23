# Rappel des commandes 
## Équipements utilisés

- **Switch cœur :** Cisco 3750 L3
- **Routeurs :** Cisco 1921 
- **Serveurs :** [À compléter]  

## Switch 

### 🔐 Configuration SSH et utilisateurs sur le switch cœur

```
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

## 🗂️ Configuration des VLAN

## Création d’un VLAN
```
Switch(config)# vlan 10
Switch(config-vlan)# name Utilisateurs
Switch(config)# vlan 20
Switch(config-vlan)# name Serveurs
```
### Attribution d’un VLAN à un port en mode Access
```
Switch(config)# interface GigabitEthernet1/0/3
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
```
### Suppression d’un VLAN 
```
Switch(config)# no vlan 20
```
### Vérification des VLAN existants
```
Switch# show vlan brief
```

## 🚦 Configuration des ports : Trunk et Access

### Mode Trunk
```
Switch(config)# interface GigabitEthernet1/0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20
Switch# show interfaces trunk
Switch# show running-config
```
### Mode Access
```
Switch(config)# interface GigabitEthernet1/0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch# show interfaces status
Switch# show running-config

```
## 🖧 Stack 
```
Switch# show version              
Switch# show switch               
Switch(config)# switch <num> priority 15   
Switch# show switch stack-ports   
Switch# write memory                        
```
## 🔗 LACP / EtherChannel
```
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

## 🌐 Routage

### Activation Routage

```
Switch(config)# ip routing
```
### Passerelle par défaut


```
Switch(config)# ip route 0.0.0.0 0.0.0.0 <IP_Gateway>
```
### Routes statiques
Ajouter des routes vers des réseaux spécifiques :  
```
Switch(config)# ip route <Réseau1> <Masque1> <Next_Hop1>
Switch(config)# ip route <Réseau2> <Masque2> <Next_Hop2>
Switch(config)# ip route <Réseau3> <Masque3> <Next_Hop3>
```
### Vérification
Vérifier les routes configurées :  
```
Switch# show ip route
```
## 🛡️ ACL 

### ACL pour autoriser le VLAN Interco
Créer une ACL pour autoriser le trafic du VLAN Interco vers Internet :  
```
Router(config)# access-list 100 permit ip 192.168.10.0 0.0.0.255 any
```
### Appliquer l’ACL sur l’interface sortante
```
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group 100 out
```
### Vérification
```
Router# show access-lists
```
## 🔀 NAT/PAT
### NAT pour traduire les adresses internes en IP publique
```
Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255
Router(config)# ip nat inside source list 1 interface GigabitEthernet0/0 overload
```
### Définir les interfaces NAT Inside / Outside
```
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat inside
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat outside
```
### Vérification
```
Router# show ip nat translations
Router# show ip nat statistics
```