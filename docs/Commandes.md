```
Coeur> enable
Coeur# show ip ssh (pour verifier si il suporte ssh)
Coeur# conf t
Coeur(config)# enable secret "MotDePasse"
Coeur(config)# ip domain name "NomDeDomaine.com"
Coeur(config)# ip ssh version 2 
Coeur(config)# crypto key generate rsa
	How many bits in the modulus [512]: 1024 
Coeur(config)# username "NomDeL'Utilisateur" privilege 15 secret "MotDePasse"
*Coeur(config)# line vty 0 15 
Coeur(config)# login local
Coeur(config)# transport input ssh 
Coeur(config)# exit
Coeur# exit
Coeur# copy run start* 
```
