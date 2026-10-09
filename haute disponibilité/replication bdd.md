- modification du fichier config mysql
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

- Vérouiller l'édition des tables SQL
```bash
sudo mysql
```
```sql
flush tables with read lock;
```

- Dévérouiller l'édition des tables sql
```bash
sudo mysql
```
```sql
unlock tables;
```

- Créer un compte réplicateur
```sql
create user 'replicateur'@'%' identified by 'Btssio2017';
```

- Donner tout les droit au compte réplicateur
```sql
grant replication slave on *.* to 'replicateur'@'%';
```

- Voir le status du master
```bash
sudo mysql
```
```sql
show master status;
```
- Voir le status du slave
```bash
sudo mysql
```
```sql
show slave status \G;
```

- Recréer le fichier log
```bash
sudo mkdir -m 2750 /var/log/mysql
sudo chown mysql /var/log/mysql
```

- Arreter le slave
```bash
sudo mysql
```
```sql
stop slave;
```

- Démarrer le slave
```bash
sudo mysql
```
```sql
start slave;
```

- Paraméter le serveur esclave pour reconnaitre le master
```bash
sudo mysql
```
```sql
change master to master_host='172.16.0.10', master_user='replicateur', master_password='Btssio2017', master_log_file='mysql-bin.000002', master_log_pos=328;
```

- Voir le status du slave
```bash
sudo mysql
```
```sql
show slave status;
```

### Réplication bi-directionel (maitre to maitre)

- modification du fichier config mysql web1
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
- Config fichier serv-web1
1. Rajouter la ligne **log-slave-updates**
2. Rajouter la ligne **master-retry-count = 20**
3. Rajouter la ligne **replicate-do-db = gsb_valide**

- Config fichier serv-web2
1. Décommentez la ligne **bind-address**
2. Décommentez la ligne **log_bin**
3. Rajouter la ligne **binlog_do_db = gsb_valide**
4. Rajouter la ligne **log-slave-updates**

- Sur serv-web2
1. Créer l'utilisateur replicateur
2. lui donner tout les droit
3. arréter le slave:

- Sur serv1
1. Entrez la commande suivavnte dans la console mysql : change master to master_host='172.16.0.11', master_user='replicateur', master_password='Btssio2017', master_log_file='mysql-bin.000001', master_log_pos=328;

- sur serv-web2
1. Entrez la commande suivante dans la console mysql : change master to master_host='172.16.0.10', master_user='replicateur', master_password='Btssio2017', master_log_file='mysql-bin.000003', master_log_pos=342;

- Sur les 2 serveur
1. slave start;
2. show slave status \G;
