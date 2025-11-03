# Réseaux présents sur l’infrastructure

## Réseau Client

Les adresses IP sont attribuées par un serveur **DHCP**, dans la plage de **10 à 50**.  
Ce réseau permet aux utilisateurs d’utiliser les ordinateurs et d’accéder aux services internes ou externes selon les règles de sécurité définies.

(Ordinateur)

| Nom du réseau| IP Réseau|Maque| Plage d’adresses| Passerelle (Gateway)|Adresse Broadcast|
|------------|----|------|--------------|----------|-----------|
|Client|172.28.96.0|/24|172.28.96.1-172.28.96.253|172.28.96.254|172.28.96.255|

---

## Réseau Serveur

Ce réseau regroupe l’ensemble des **serveurs internes** de l’infrastructure.  
Les adresses IP y sont configurées en **statique** afin de garantir une accessibilité constante des services hébergés.

(Serveur AD,Serveur DNS resolveur)

| Nom du réseau| IP Réseau|Maque| Plage d’adresses| Passerelle (Gateway)|Adresse Broadcast|
|------------|----|------|--------------|----------|-----------|
|Serveur|172.28.120.0|/24|172.28.120.1-172.28.120.253|172.28.120.254|172.28.120.255|

---

## Réseau Management

Ce réseau est dédié à la **gestion des équipements** de l’infrastructure (switchs, pare-feu, routeurs, etc.).  
Les adresses IP sont également configurées en **statique** pour assurer un contrôle fiable et sécurisé.

(Switch,routeur,pare-feu,ordinateur,machine virtuel,serveur)

| Nom du réseau| IP Réseau|Maque| Plage d’adresses| Passerelle (Gateway)|Adresse Broadcast|
|------------|----|------|--------------|----------|-----------|
|Management|192.168.140.0|/24|192.168.140.1-192.168.140.253|192.168.140.254|192.168.140.255|

---
## Réseau DMZ public

Le réseau **DMZ (Demilitarized Zone) public** héberge les services accessibles depuis l’extérieur (serveur web, mail, etc.) tout en isolant ces derniers du réseau interne.  
Les adresses IP y sont également en **statique**.

(Serveur DNS d'autorité)
(Serveur web)

| Nom du réseau| IP Réseau|Maque| Plage d’adresses| Passerelle (Gateway)|Adresse Broadcast|
|------------|----|------|--------------|----------|-----------|
|DMZ public|192.168.45.0|/24|192.168.45.1-192.168.45.253|192.168.45.254|192.168.45.255|


---
## Réseau DMZ privée

Le réseau **DMZ (Demilitarized Zone) privé** héberge les services accessibles depuis la DMZ public (base de données etc.) tout en isolant ces derniers du réseau interne.  
Les adresses IP y sont également en **statique**.

(Base de données)

| Nom du réseau| IP Réseau|Maque| Plage d’adresses| Passerelle (Gateway)|Adresse Broadcast|
|------------|----|------|--------------|----------|-----------|
|DMZ privée|192.168.55.0|/24|192.168.55.1-192.168.55.253|192.168.55.254|192.168.55.255|


---
## Réseau LAN to DMZ

Ce réseau assure l’**interconnexion entre le switch cœur de réseau et le pare-feu virtuel**.  
Les adresses IP sont définies en **statique** pour garantir la stabilité des flux entre le LAN et la DMZ.

| Nom du réseau| IP Réseau|Maque| Plage d’adresses| Passerelle (Gateway)|Adresse Broadcast|
|------------|----|------|--------------|----------|-----------|
|LANtoDMZ|10.0.0.0|/25|10.0.0.1-10.0.0.125|10.0.0.126|10.0.0.127|

---

## Réseau DMZ to WAN

Ce réseau assure l’**interconnexion entre le pare-feu physique et les routeurs** d’accès Internet.  
Les adresses IP sont configurées en **statique** afin d’assurer une communication stable vers l’extérieur.

| Nom du réseau| IP Réseau|Maque| Plage d’adresses| Passerelle (Gateway)|Adresse Broadcast|
|------------|----|------|--------------|----------|-----------|
|LANtoDMZ|10.0.0.128|/25|10.0.0.129-10.0.0.253|10.0.0.254|10.0.0.255|

---



