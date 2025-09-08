# 🌐 Adressage IP des Sites

Répartition des adresses IP par groupe géographique :

| 📍 **Site**     | 🧭 **Plage d'adresses**         | 🛰️ **CIDR**         |
|----------------|----------------------------------|---------------------|
| **Blois**      | 172.28.0.0 → 172.28.31.0         | `172.28.0.0/19`     |
| **Chartres**   | 172.28.32.0 → 172.28.63.0        | `172.28.32.0/19`    |
| **Tours**      | 172.28.64.0 → 172.28.95.0        | `172.28.64.0/19`    |
| **Orléans (Nous)** | 172.28.96.0 → 172.28.127.0  | `172.28.96.0/19`    |
| **Bourges**    | 172.28.128.0 → 172.28.159.0      | `172.28.128.0/19`   |

> 💡 Chaque site dispose d’une plage de **8 192 adresses IP** (car `/19` = 2¹³).

---

🎯 **Remarques :**
- Les plages sont continues, sans chevauchement.
- Orléans correspond à **notre site**.
- Ces adresses peuvent être subdivisées selon les besoins internes (réseaux VLAN, DMZ, etc).

| Sites | Plages d’adresses | CIDR |
| --- | --- | --- |
| Orléans | 172.28.96.0 → 172.28.127.0 | 172.28.96.0 /19 |
|  |  |  |