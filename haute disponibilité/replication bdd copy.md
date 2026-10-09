### Sur le serveur maitre
# Modifier le fichier de config 50-server.cnf
```bash
sudo cp /etc/mysql/mariadb.conf.d/50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf.sav
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
- Modifier les option suivante
```bash
bind-address = <ip de la machine>
log_error = /var/log/mysql/error.log
server-id = 100
log_bin = /var/log/mysql/mysql-bin.log
expire_logs_days = 10
max_bindlog_size = 100M
binlog_do_db = <nom de la base de données>
```
# Créer le dossier d'erreur mysql et définir les permissions
```bash
sudo mkdir -m 2750 /var/log/mysql
```

# Redémarrer le service mysql
```bash
sudo systemctl restart mariadb.service
```
# Vérifier si il n'y a pas d'erreur
```bash
systemctl status mariadb.service
```

### Sur le serveur esclave
# Modifier le fichier de config 50-server.cnf
```bash
sudo cp /etc/mysql/mariadb.conf.d/50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf.sav
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
- Modifier les option suivante
```bash
log_error = /var/log/mysql/error.log
server-id = 101
expire_log_days = 10
max_binlog_size = 100M
master-retry-count = 20
binlog_do_db = <nom de la base de données>
```
# Créer le dossier d'erreur mysql
```bash
sudo mkdir -m 2750 /var/log/mysql
```

# Redémarrer le service mysql
```bash
sudo systemctl restart mariadb.service
```
# Vérifier si il n'y a pas d'erreur
```bash
systemctl status mariadb.service
```
# Création de l'utilisateur pour la réplication (sur les 2 machines)
```sql
create user 'replicateur'@'%' identified by 'Btssio2017';
```

- Donner tout les droit au compte réplicateur
```sql
grant replication slave on *.* to 'replicateur'@'%';
flush privileges;
```

### Sur la machine master
# Bloquer l'écriture dans la base de données
```bash
sudo mysql
```
```sql
flush tables with read lock;
```
# Récupérer les info du master
```bash
sudo mysql
```
```sql
show master status;
```
- Récupérer le **file** (ex : mysql-bin.000001) et la **Position** (ex : 3921)

### Sur la machine slave
# Configuration de mysql pour se connecter au master
```bash
sudo mysql
```
```sql
change master to master_host='ip du master', master_user='replicateur', master_password='Btssio2017', master_log_file='mysql-bin.000002', master_log_pos=328;
```
### Sur le serveur maitre
# Dévérouillage des tables
```bash
sudo mysql
```
```sql
unlock tables;
```

### Sur la machine slave
# Vérifier le status de l'esclave
```bash
sudo mysql
```
```sql
show slave status \G;
```
- Vous devez voir s'afficher **Waiting for master to send evant** a l'onglet **Slave_IO_State**

### Passer le master slave à master master
# Création de la ressource
```bash
sudo crm configure primitive serviceMySQL ocf:heartbeat:mysql
```

# Cloner la ressource
```bash
crm configure clone cServiceMySQL serviceMySQL
```

# Vérification du clone
```bash
sudo crm status
```

# modification du fichier config mysql sur serveur1
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
# Config fichier serv-web1
1. Rajouter la ligne **log-slave-updates**
2. Rajouter la ligne **master-retry-count = 20**
3. Rajouter la ligne **replicate-do-db = gsb_valide**

# modification du fichier config mysql sur serveur2
1. Décommentez la ligne **bind-address**
2. Décommentez la ligne **log_bin**
3. Rajouter la ligne **binlog_do_db = gsb_valide**
4. Rajouter la ligne **log-slave-updates**

# Sur serveur2
# Création de l'utilisateur pour la réplication
```sql
create user 'replicateur'@'%' identified by 'Btssio2017';
```
# lui donner tout les droit
```sql
grant replication slave on *.* to 'replicateur'@'%';
```
# arréter le slave
```bash
sudo mysql
```
```sql
stop slave;
```

# Sur le serveur1
# Entrez la commande suivavnte dans la console mysql : 
```bash
sudo mysql
```
```sql
change master to master_host='172.16.0.11', master_user='replicateur', master_password='Btssio2017', master_log_file='mysql-bin.000001', master_log_pos=328;
```

# sur le serveur2
# Entrez la commande suivante dans la console mysql : 
```bash
sudo mysql
```
```sql
change master to master_host='172.16.0.10', master_user='replicateur', master_password='Btssio2017', master_log_file='mysql-bin.000003', master_log_pos=342;
```

- Sur les 2 serveur
# Alluler le slave
```bash
sudo mysql
```
```sql
start slave;
```

2. show slave status \G;

- Sur les 2 serveur
# Alluler le master
```bash
sudo mysql
```
```sql
start master;
```
