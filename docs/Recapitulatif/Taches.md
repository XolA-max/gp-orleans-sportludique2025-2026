# Compte rendu des séances

## 📅 Semaine 1

| Tâches                                  | Descriptions                                                                 | Personnes concernées       |
|-----------------------------------------|----------------------------------------------------------------------------|----------------------------|
| Mise en place de la configuration du Switch ORL-SW-CORE | Brancher le switch avec le câble console sur un ordinateur<br>- Hostname<br>- SSH<br>- Adresse IP | Antoine, Cyriak, Louis     |
| Ajout d’un utilisateur local `sio2orl`  | Ajout de cet utilisateur pour pouvoir changer la configuration de la carte réseau et la mettre sur le réseau  | Antoine, Louis, Cyriak     |
| Stack des deux Switch                   | - Brancher le câble<br>- Vérification de la connexion en branchant le câble sur les deux switches | Antoine, Cyriak, Louis     |
|Création des Vlans        |-Vlan Managments<br>-Vlan Client<br>-Vlan Serveur              |Antoine,Louis,Cyriak                      |

---

## 📅 Semaine 2

| Tâches                       | Descriptions                                                                 | Personnes concernées       |
|------------------------------|----------------------------------------------------------------------------|----------------------------|
| Création AD                  | Configuration DNS et Nom de domaine                                         | Cyriak|
|Switch coeur|Finalisation de configuration du switch coeur             |Antoine,Louis                    |
|Routeur fibre        |Ajout du routeur fibre             |Antoine,Louis
|Création Vlan       |-Vlan Interco           |Antoine   |

---

## 📅 Semaine 3

| Tâches | Descriptions | Personnes concernées |
|--------|--------------|----------------------|
|Routeur ADLS|Ajout d’un deuxième routeur ADSL qui prendra le relais si le routeur fibre rencontre une défaillance ou si les connexions sont débranchées.              |Cyriak,Louis|
|Switch salle serveur|Configuration du switch situé dans la salle des serveurs              |Louis,Cyriak,Antoine                      |
|Documentation |Mise à jour de la documentation              |Antoine|
|HA|Mise en place de HA pour crée une passerelle virtuel              |Cyriak,Louis,Antoine

---

## 📅 Semaine 4

| Tâches | Descriptions | Personnes concernées |
|--------|--------------|----------------------|
|Documentation |Mise à jour de la documentation|Antoine
|Raid 5|Instalation Raid 5 sur le serveur | Louis,Cyriak |
|Proxmox|Instalation de proxmox sur notre serveur(Pas effectué) | Louis,Cyriak |
|Parefeu|Reflexion parfeu | Antoine,Louis,Cyriak |

---

## 📅 Semaine 5

| Tâches | Descriptions | Personnes concernées |
|--------|--------------|----------------------|
| Proxmox            | Installation de Proxmox sur notre serveur                          | Louis                 |
| Infrastructure     | Modification des routeurs et switchs pour accueillir le pare-feu   | Antoine               |
| Pare-feu physique  | Configuration du pare-feu et mise en place dans la baie            | Antoine, Louis, Cyriak|
| OPNsense           | Réflexion sur la mise en place de la DMZ et d’un pare-feu virtuel  | Cyriak, Louis         |

---

## 📅 Semaine 6

| Tâches | Descriptions | Personnes concernées |
|--------|--------------|----------------------|
|OPNsense|Mise en place de OPNsense sur le serveur nutanix|Cyriak,Antoine|
|Infrastructure |Modification des routeurs et switchs pour accueillir le  deuxime pare-feu |Louis,Antoine|
|DNS|Resolver|Cyriak|
|DNS|Autorité|Louis|

---

## 📅 Semaine 7

| Tâches | Descriptions | Personnes concernées |
|--------|--------------|----------------------|
|DNS|Autorité|Louis, Antoine|
|AD|Ajout utilisateur de l'ad|Antoine, Cyriak|
|DHCP|Modification du DHCP sur l'active directory|Antoine, Cyriak|
|Routeur|Ajout du PAT sur les deux routeurs|Antoine|
|Serveur web|Apache|Louis|

---

## 📅 Semaine 8

| Tâches | Descriptions | Personnes concernées |
|--------|--------------|----------------------|
|DMZ privée|Création de la DMZ privée|Cyriak,Antoine|
|Serveur web|Apache|Louis|
|Serveur Web|Assesibilité au serveur web depuis l'interieur et exterieur|Antoine|
