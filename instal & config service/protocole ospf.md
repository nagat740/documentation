### Mise en place du protocole OSPF

## Le protocole OSPF permet de faire du routage dynamique sur un routeur à la manière du protocole RIP

## Dans la console du routeur en mode configuration (conf t) :
- Activation de OSPF
```cisco
routeur ospf <numéro de processus>
ex : routeur OSPF 1
```

- Nom du routeur (optionnel, OSP choisira lui-même le nom si commande non définie)
```cisco
routeur-id <ip ou nom>
ex : routeur-id 172.16.0.254
```

- Déclaration des réseaux connectés au routeur
```cisco
network <ip du réseau> <masque inversé> area <numéro de zone>
ex : network 172.16.0.0 0.0.255.255 area 0
```

- Déclaration d'une interface passive
```cisco
passive-interface <interface routeur voulut>
ex : passive-interface fa0/1
```

- Vérification du voisinage d’un routeur (sortir du mode config)
```cisco
show ip ospf neighbor
```

- Observation des messages OSPF (sortir du mode config)
```cisco
Debug ip ospf events
```
