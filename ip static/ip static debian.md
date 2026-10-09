```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens33
iface ens33 inet static
        address 172.16.0.20/24
        gateway 172.16.0.254
        dns-nameservers 8.8.8.8
        dns-search sodecaf.local
```
