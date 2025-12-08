# Compte rendu des séances

## 📅 Semaine 1

| Tâches                                  | Descriptions                                                                 | Personnes concernées       |
|-----------------------------------------|----------------------------------------------------------------------------|----------------------------|
| Mise en place de la configuration du Switch ORL-SW-CORE | Brancher le switch avec le câble console sur un ordinateur<br>- Hostname<br>- SSH<br>- Adresse IP | Antoine, Cyriak, Louis     |
| Ajout d'un utilisateur local `sio2orl`  | Ajout de cet utilisateur pour pouvoir changer la configuration de la carte réseau et la mettre sur le réseau  | Antoine, Louis, Cyriak     |
| Stack des deux Switch                   | - Brancher le câble<br>- Vérification de la connexion en branchant le câble sur les deux switches | Antoine, Cyriak, Louis     |
| Création des VLANs                      | - VLAN Management<br>- VLAN Client<br>- VLAN Serveur              | Antoine, Louis, Cyriak     |

---

## 📅 Semaine 2

| Tâches                       | Descriptions                                                                 | Personnes concernées       |
|------------------------------|----------------------------------------------------------------------------|----------------------------|
| Création AD                  | Configuration DNS et nom de domaine                                         | Cyriak                     |
| Switch cœur                  | Finalisation de configuration du switch cœur                                | Antoine, Louis             |
| Routeur fibre                | Ajout du routeur fibre                                                      | Antoine, Louis             |
| Création VLAN                | - VLAN Interco                                                              | Antoine                    |

---

## 📅 Semaine 3

| Tâches                | Descriptions                                                                                                                                                  | Personnes concernées  |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|
| Routeur ADSL          | Ajout d'un deuxième routeur ADSL qui prendra le relais si le routeur fibre rencontre une défaillance ou si les connexions sont débranchées                   | Cyriak, Louis         |
| Switch salle serveur  | Configuration du switch situé dans la salle des serveurs                                                                                                      | Louis, Cyriak, Antoine|
| Documentation         | Mise à jour de la documentation                                                                                                                               | Antoine               |
| HA                    | Mise en place de HA pour créer une passerelle virtuelle                                                                                                       | Cyriak, Louis, Antoine|

---

## 📅 Semaine 4

| Tâches        | Descriptions                                            | Personnes concernées  |
|---------------|--------------------------------------------------------|-----------------------|
| Documentation | Mise à jour de la documentation                         | Antoine               |
| RAID 5        | Installation RAID 5 sur le serveur                      | Louis, Cyriak         |
| Proxmox       | Installation de Proxmox sur notre serveur (pas effectué)| Louis, Cyriak         |
| Pare-feu      | Réflexion pare-feu                                      | Antoine, Louis, Cyriak|

---

## 📅 Semaine 5

| Tâches             | Descriptions                                                           | Personnes concernées  |
|--------------------|-----------------------------------------------------------------------|-----------------------|
| Proxmox            | Installation de Proxmox sur notre serveur                              | Louis                 |
| Infrastructure     | Modification des routeurs et switchs pour accueillir le pare-feu       | Antoine               |
| Pare-feu physique  | Configuration du pare-feu et mise en place dans la baie                | Antoine, Louis, Cyriak|
| OPNsense           | Réflexion sur la mise en place de la DMZ et d'un pare-feu virtuel     | Cyriak, Louis         |

---

## 📅 Semaine 6

| Tâches         | Descriptions                                                                  | Personnes concernées |
|----------------|------------------------------------------------------------------------------|----------------------|
| OPNsense       | Mise en place de OPNsense sur le serveur Nutanix                             | Cyriak, Antoine      |
| Infrastructure | Modification des routeurs et switchs pour accueillir le deuxième pare-feu    | Louis, Antoine       |
| DNS            | Resolver                                                                      | Cyriak               |
| DNS            | Autorité                                                                      | Louis                |

---

## 📅 Semaine 7

| Tâches   | Descriptions                                         | Personnes concernées |
|----------|-----------------------------------------------------|----------------------|
| DNS      | Autorité                                             | Louis, Antoine       |
| AD       | Ajout utilisateurs de l'AD                           | Antoine, Cyriak      |
| DHCP     | Modification du DHCP sur l'Active Directory          | Antoine, Cyriak      |
| Routeur  | Ajout du PAT sur les deux routeurs                   | Antoine              |
| Serveur web | Apache                                            | Louis                |

---

## 📅 Semaine 8

| Tâches          | Descriptions                                                                   | Personnes concernées  |
|-----------------|-------------------------------------------------------------------------------|-----------------------|
| DMZ privée      | Création de la DMZ privée                                                      | Cyriak, Antoine       |
| Serveur web     | Apache                                                                         | Louis                 |
| Serveur Web     | Accessibilité au serveur web depuis l'intérieur et l'extérieur                | Antoine               |
| Certificat SSL  | Serveur web sécurisé avec le protocole SSL                                     | Cyriak, Louis, Antoine|
| Reverse Proxy   | Proxy pour avoir plusieurs sites web disponibles depuis l'extérieur           | Antoine               |

---

## 📅 Semaine 9

| Tâches                  | Descriptions                                   | Personnes concernées  |
|-------------------------|-----------------------------------------------|-----------------------|
| Serveur Base de données | Base de données pour WordPress                 | Antoine               |
| Serveur WordPress       | WordPress                                      | Antoine               |
| Certificat SSL          | Serveur web sécurisé avec le protocole SSL     | Cyriak, Louis         |

---

## 📅 Semaine 10

| Tâches                       | Descriptions                            | Personnes concernées |
|------------------------------|-----------------------------------------|----------------------|
| Proxmox                      | Installation des VM et configuration réseaux | Antoine, Cyriak      |
| DNS-resolver-sec             | Résilience du service DNS               | Cyriak               |
| Serveur mail                 | Création du serveur mail dans la DMZ    | Louis                |
| DNS-autorité-sec             | Résilience du service DNS               | Cyriak               |
| Active Directory secondaire  | Résilience du service Active Directory  | Antoine              |

---

## 📅 Semaine 11

| Tâches                       | Descriptions                            | Personnes concernées |
|------------------------------|-----------------------------------------|----------------------|
| Active Directory secondaire  | Résilience du service Active Directory  | Antoine              |
| Serveur mail                 | Création du serveur mail dans la DMZ    | Louis                |
| Règles Pare-feu              | Règles de filtrage du Stormshield       | Antoine              |
| DNS-autorité-sec             | Résilience du service DNS               | Cyriak               |

---

## 📅 Semaine 12

| Tâches           | Descriptions                            | Personnes concernées |
|------------------|-----------------------------------------|----------------------|
| DNS-autorité-sec | Résilience du service DNS               | Cyriak               |
| Serveur mail     | Création du serveur mail dans la DMZ    | Louis                |
| Règles Pare-feu  | Règles de filtrage de OPNsense          | Antoine              |

---

## 📅 Semaine 13

| Tâches                  | Descriptions                                  | Personnes concernées |
|-------------------------|----------------------------------------------|----------------------|
| Graylog                 | Serveur de log centralisé                     | Cyriak               |
| GLPI                    | Serveur GLPI                                  | Louis                |
| Règles firewall OPNsense| Règles firewall                               | Antoine              |
| Users GLPI              | Ajout des utilisateurs de l'AD sur GLPI       | Antoine              |
| Win clients             | Win clients sur le domaine                    | Antoine              |