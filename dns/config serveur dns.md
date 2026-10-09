# Fichier config bind9 (/etc/bind/)
- named.conf.local = fichier pour définir des zones

# Déclaration d'une zone sur un serveur maitre
```bash
zone "sodecaf.fr" {
        type master;
        file "db.sodecaf.fr";
        allow-transfer {172.16.0.4;};
};
```

# /var/cache/bind/db.sodecaf.fr (fichier de zone)
```bash
$TTL 86400
@       IN      SOA     srv-dns1.sodecaf.fr. hostmaster.sodecaf.fr. (

                        2008113001 ;serial
                        86400 ;refresh
                        21600 ;retry
                        3600000 ;expire
                        3600 ) ;negative caching ttl

@       IN      NS      srv-dns1.sodecaf.fr.
@       IN      NS      srv-dns2.sodecaf.fr.

srv-dns1        IN      A       172.16.0.3
srv-dns2        IN      A       172.16.0.4
srv-web1        IN      A       172.16.0.10
srv-web2        IN      A       172.16.0.11
www             IN      A       172.16.0.12
web1            IN      CNAME   srv-web1.sodecaf.fr.
web2            IN      CNAME   srv-web2.sodecaf.fr.
```

# Zone dns reverse
```bash
zone "0.16.172.in-addr.arpa" {
    type master;
    file "db.172.16.0.rev";
    allow-transfer {172.16.0.4;};
};
```
# named.conf.local (dns secondaire)
```bash
zone "sodecaf.fr" {
        type slave;
        file "slave/db.sodecaf.fr";
        masters {172.16.0.3;};
};

zone "0.16.172.in-addr.arpa" {
        type slave;
        file "slave/db.172.16.0.rev";
         masters {172.16.0.3;};
};
```

# Commande nslookup
- nslookup {nom à tester} {ip du serveur dns}