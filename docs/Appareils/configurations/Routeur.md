
## Routeurs

###  Routage

#### Activation Routage

!!! info
    ```bash
    Switch(config)# ip routing
    ```

#### Passerelle par défaut

!!! info
    ```bash
    Switch(config)# ip route 0.0.0.0 0.0.0.0 <IP_Gateway>
    ```

#### Routes statiques

Ajouter des routes vers des réseaux spécifiques :  

!!! info
    ```bash
    Switch(config)# ip route <Réseau1> <Masque1> <Next_Hop1>
    Switch(config)# ip route <Réseau2> <Masque2> <Next_Hop2>
    Switch(config)# ip route <Réseau3> <Masque3> <Next_Hop3>
    ```

#### Vérification

Vérifier les routes configurées :  

!!! info
    ```bash
    Switch# show ip route
    ```

###  Encapsulation Dot1Q

### Configuration d’une sous-interface pour le routage inter-VLAN

Configurer une interface routeur pour transporter plusieurs VLANs via **802.1Q** :  

!!! info
    ```bash
    Router(config)# interface GigabitEthernet0/0.10
    Router(config-subif)# encapsulation dot1Q 10
    Router(config-subif)# ip address 192.168.10.1 255.255.255.0
    
    Router(config)# interface GigabitEthernet0/0.20
    Router(config-subif)# encapsulation dot1Q 20
    Router(config-subif)# ip address 192.168.20.1 255.255.255.0
    ```

###  ACL

#### ACL pour autoriser le VLAN Interco

Créer une ACL pour autoriser le trafic du VLAN Interco vers Internet :  

!!! info
    ```bash
    Router(config)# access-list 100 permit ip 192.168.10.0 0.0.0.255 any
    ```

#### Appliquer l’ACL sur l’interface sortante

!!! info
    ```bash
    Router(config)# interface GigabitEthernet0/0
    Router(config-if)# ip access-group 100 out
    ```

#### Vérification

!!! info
    ```bash
    Router# show access-lists
    ```

###  NAT

#### NAT pour traduire les adresses internes en IP publique

!!! info
    ```bash
    Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255
    Router(config)# ip nat inside source list 1 interface GigabitEthernet0/0 overload
    ```

#### Définir les interfaces NAT Inside / Outside

!!! info
    ```bash
    Router(config)# interface GigabitEthernet0/1
    Router(config-if)# ip nat inside
    Router(config)# interface GigabitEthernet0/0
    Router(config-if)# ip nat outside
    ```

#### Vérification

!!! info
    ```bash
    Router# show ip nat translations
    Router# show ip nat statistics
    ```

###  HSRP (Hot Standby Router Protocol)

#### Configuration sur le premier routeur (Routeur FIBRE)

!!! info
    ```bash
    RouterA(config)# interface GigabitEthernet0/1
    RouterA(config-if)# ip address 192.168.1.2 255.255.255.0
    RouterA(config-if)# standby 1 ip 192.168.1.1
    RouterA(config-if)# standby 1 priority 110
    RouterA(config-if)# standby 1 preempt
    ```

#### configuration sur le second routeur (Routeur ADSL)

!!! info
    ```bash
    RouterB(config)# interface GigabitEthernet0/1
    RouterB(config-if)# ip address 192.168.1.3 255.255.255.0
    RouterB(config-if)# standby 1 ip 192.168.1.1
    RouterB(config-if)# standby 1 priority 100
    RouterB(config-if)# standby 1 preempt
    ```

### PAT

!!! info
    ```bash
    ip nat inside source static udp 192.168.45.2 53 interface g0/1 53
    ip nat inside source static tcp 192.168.45.2 53 interface g0/1 53
    ```
