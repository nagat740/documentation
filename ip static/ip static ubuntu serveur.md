# Installation vlan
```bash
apt-get update -y
apt-get upgrade -y
apt-get install vlan
```

# Activation du module
```bash
echo "8021q" | sudo tee -a /etc/modules
```

# Application du module
```bash
sudo modprobe 8021q
```

# Vérification du module
```bash
lsmod | grep 8021q
```
- Vous devriez voir cela
```bash
8021q                  40960  0
garp                   16384  1 8021q
mrp                    20480  1 8021q
```

## Création du fichier de configuration ip
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
## contenue du fichier de configuration
```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 172.16.0.6/24
      routes:
        - to: default
          via: 172.16.0.254
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```
