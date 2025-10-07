# Réseaux présents sur l’infrastructure

## Réseau Client

Les adresses IP sont attribuées par un serveur **DHCP**, dans la plage de **10 à 50**.  
Ce réseau permet aux utilisateurs d’utiliser les ordinateurs et d’accéder aux services internes ou externes selon les règles de sécurité définies.

---

## Réseau Serveur

Ce réseau regroupe l’ensemble des **serveurs internes** de l’infrastructure.  
Les adresses IP y sont configurées en **statique** afin de garantir une accessibilité constante des services hébergés.

---

## Réseau Management

Ce réseau est dédié à la **gestion des équipements** de l’infrastructure (switchs, pare-feu, routeurs, etc.).  
Les adresses IP sont également configurées en **statique** pour assurer un contrôle fiable et sécurisé.

---

## Réseau LAN to DMZ

Ce réseau assure l’**interconnexion entre le switch cœur de réseau et le pare-feu virtuel**.  
Les adresses IP sont définies en **statique** pour garantir la stabilité des flux entre le LAN et la DMZ.

---

## Réseau DMZ

Le réseau **DMZ (Demilitarized Zone)** héberge les services accessibles depuis l’extérieur (serveur web, mail, etc.) tout en isolant ces derniers du réseau interne.  
Les adresses IP y sont également en **statique**.

---

## Réseau DMZ to WAN

Ce réseau assure l’**interconnexion entre le pare-feu physique et les routeurs** d’accès Internet.  
Les adresses IP sont configurées en **statique** afin d’assurer une communication stable vers l’extérieur.
