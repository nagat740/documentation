# signer une zone

- Modifier fichier /etc/bind/named.conf.options pour remplacer dnssec-validation auto; par dnssec-validation yes;

- Créer une paire de clé
```bash
sudo dnssec-keygen -a rsasha1 -b 1024 -n zone sodecaf.fr
sudo dnssec-keygen -a rsasha1 -b 1024 -f KSK -n zone sodecaf.fr
```
- Signez une zone
```bash
sudo dnssec-signzone -o sodecaf.fr -t -k /etc/bind/keys/Ksodecaf.fr.+005+19891 db.sodecaf.fr /etc/bind/keys/Ksodecaf.fr.+005+48556
```

- Rajouter ces 2 lignes à la fin du fichier /var/cache/bind/db.sodecaf.fr
```bash
; KSK
$include "/etc/bind/keys/Ksodecaf.fr.+005+19891.key"

; ZSK
$include "/etc/bind/keys/Ksodecaf.fr.+005+48556.key"
```

- Modifier le fichier /etc/bind/named.conf.local et changer le fichier utiliser dans file de db.sodecaf.fr par db.sodecaf.fr.signed