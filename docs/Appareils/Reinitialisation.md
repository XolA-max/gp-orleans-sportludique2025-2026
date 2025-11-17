# Guide : Réinitialisation et ROMMON sur Routeurs Cisco

### Conditions préalables

    Accès privilégié (enable mode) sur le routeur.

    Connexion console pour ROMMON ou récupération si le mot de passe est perdu.

    Prendre en compte que ces opérations effacent totalement la configuration, sauf certains paramètres ROMMON (warm-reboot, memory-size iomem).

 Réinitialisation du routeur Cisco

Deux méthodes principales sont disponibles.
## Méthode 1 — Réinitialisation standard (registre 0x2102)

#### Vérifier le registre de configuration
```
Router# show version
```

>La ligne doit indiquer : ```Configuration register is 0x2102```


Si nécessaire, corriger :
```

Router# configure terminal
Router(config)# config-register 0x2102
Router(config)# end
```

#### Effacer la configuration
```


Router# write erase
```

#### Redémarrer
```

Router# reload
System configuration has been modified. Save? [yes/no]: n
```
 > Au redémarrage, répondre no au dialogue : 
```

Would you like to enter the initial configuration dialog? [yes/no]:n
```

---

## Méthode 2 — Réinitialisation via registre 0x2142 (ignorer la configuration existante)

#### Changer le registre
```

Router# configure terminal
Router(config)# config-register 0x2142
Router(config)# end
```

#### Redémarrer
```

Router# reload
System configuration has been modified. Save? [yes/no]: n
```

>Au redémarrage, répondre no au System Configuration Dialog.

#### Rétablir le registre standard
```

Router# configure terminal
Router(config)# config-register 0x2102
Router(config)# end
```

#### Écraser la configuration existante
```

Router# write memory
```

>Redémarrer à nouveau
```
Router# reload
```

## Le routeur est maintenant complètement restauré aux paramètres d’usine

---

##  Accéder au mode ROMMON

Le ROMMON est un mode bas niveau pour dépannage, récupération de mot de passe et restauration IOS.

---

### 1. Préparer la connexion console

> Câble console : RJ45 → USB / DB9

>Logiciel terminal : PuTTY, TeraTerm, SecureCRT
Paramètres : 9600 baud, 8 bits, pas de parité, 1 bit d’arrêt, pas de contrôle de flux 


### 2. Redémarrer le routeur
```

Router# reload
```

>ou couper et remettre l’alimentation.
### 3. Interrompre le boot

> / ! \ Dès l’apparition du message :
Use BREAK or ESC to interrupt boot process
```
Appuyer sur BREAK (Ctrl + Pause/Break) 
ou (Ctrl + Fn + B) 
ou ESC selon le clavier.
```
#### Cas particuliers

    Touche BREAK ne fonctionne pas : utiliser le menu “Send Break” de votre terminal.

    Mot de passe perdu et no service password-recovery activé : ROMMON peut être inaccessible.

### 4. Vérification

L’invite doit apparaître :
```
rommon 1 >
```



## Vérifications après réinitialisation

### Vérifier la configuration courante
```
Router# show running-config
```
> /!\ Doit être minimale (pas d’IP, pas de hostname, pas de configuration spécifique).

### Vérifier le registre
```
Router# show version
```
>Doit afficher : ```Configuration register is 0x2102```

## Commandes ROMMON utiles
```
Voir le registre:	confreg
Modifier le registre:	confreg 0x2142
Charger IOS manuellement:	boot flash:xxx.bin
Lister fichiers:	dir flash
Redémarrer:	reset
```