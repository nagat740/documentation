# Génération de la clé cluster
```bash
corosync-keygen
```
# Vérification de la clé cluster
```bash
ls -l /etc/corosync/
```

# copie du fichier d'origine de corosync pour modification
```bash
mv /etc/corosync/corosync.conf /etc/corosync/corosync.conf.ori
```
# Recréer un fichier de config
```bash
nano /etc/corosync/corosync.conf
```
- puis copier ceci dans le dossier
```bash
totem {
version: 2
cluster_name: cluster_web
crypto_cipher: aes256
crypto_hash: sha1
clear_node_high_bit:yes
}
logging {
fileline: off
to_logfile: yes
logfile: /var/log/corosync/corosync.log
to_syslog: no
debug: off
timestamp: on
logger_subsys {
subsys: QUORUM
debug: off
}
}
quorum {
provider: corosync_votequorum
expected_votes: 2
two_nodes: 1
}
nodelist {
node {
name: serv1
nodeid: 1
ring0_addr: 172.16.0.10
}
node {
name: serv2
nodeid: 2
ring0_addr: 172.16.0.11
}
}
service {
ver: 0
name: pacemaker
}
```

# Vérification de la configuration
```bash
corosync-cfgtool -s
```

# Vérification du service corosync
```bash
sudo systemctl status corosync
```

# Suivis du service corosync
```bash
sudo crm_mon
```

# Affichage de la config corosync
```bash
sudo crm configure show
```

# Désactivation de stonight
```bash
crm configure property stonith-enabled=false
```

# Désactivation de Quorum
```bash
crm configure property no-quorum-policy="ignore".
```

### Dexième partie

# Config IPFailover & serviceWeb
```bash
sudo crm configure primitive serviceWeb ocf:heartbeat:apache
```

# Configuration de l'ip virtuel
```bash
crm configure primitive IPFailover ocf:heartbeat:IPaddr2 params ip=172.16.0.12
cidr_netmask=24 nic=eth0 iflabel=VIP
```

# Constater les modifications
```bash
sudo crm_mon
ip addr show
```
# éditer une ressource
```bash
sudo crm configure edit <IPFailover>
```

# Définir le serveur primaire
```bash
crm resource move IPFailover serv1
```

# Voir la config
```bash
sudo crm configure show
```

# Teste du failover
```bash
sudo crm node standby
```
- Vérification sur le serv 2 avec crm configure show

# Réactiver serv
```bash
sudo crm node online
```

# Entrer en mode édition
```bash
sudo crm configure

configure# primitive serviceWeb lsb:apache2 op monitor interval=60s op start interval=0
timeout=60s op stop interval=0 timeout=60s

configure# commit

configure# quit
```

# Vérification de l'étape précédante
```bash
sudu crm configure show
```

# Vérifiacation de la ressousce
```bash
sudo crm_mon
```

# Regrouper les noeud
```bash
sudo crm configure

configure# group servweb IPFailover serviceWeb meta migration-threshold="5"

configure# commit
configure# quit
```

# Vérification de l'étape précédante
```bash
sudo crm configure show
```

