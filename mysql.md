# Mémo SQL
- Créer une base de données (console sql)
```sql
create database gsb_valide;
```

- Créer un nouvel utilisateur (console sql)
```sql
create user 'userGsb'@'localhost' identified by 'secret';
```

- importer un fichier .sql (console bash)
```bash
mysql gsb_valide < nom_fichier.sql
```

- Donner des droits d'un utilisateur à une base (console sql)
```sql
grant all privileges on gsb_valide.* to 'userGsb'@'localhost';
flush privileges;
```
- Afficher le contenu d'une table (console sql)
```sql
select * from Visiteur;
```

- Utilisé une base (console sql)
```sql
Use gsb_valide;
```

- Modifier un champs dans une table sql
```sql
update gsb_valide.Visiteur set mdp ='toto' where login='agest';
```

- Créer ressource mysql pour réplication avec corosync
```bash
crm configure primitive serviceMySQL ocf:heartbeat:mysql params socket=/var/run/mysqld/mysqld.sock
```

- Créer un groupe clone master to master corosync
```bash
crm configure clone cServiceMySQL serviceMySQL
```