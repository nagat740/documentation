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
# Config fichier /etc/network/interfaces
```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens32
iface ens32 inet manual

# Interface vlan 100
auto ens32.100
iface ens32.100 inet static
        address 192.168.100.9/24
        gateway 192.168.100.254
        dns-search gsb.local
        dns-nameservers 192.168.100.1 192.168.100.2
        vlan-raw-device ens32
```
# Redémarrer le service networking pour appliquer
```bash
sudo systemctl restart networking
```
