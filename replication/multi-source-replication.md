# Multi-Source-Replication 

## Background 

  * Aggregate multiple sources into one slave 
  * Uses channels (FOR CHANNEL 'replicant-1')

## Walkthrough (replicant-1)

```
-> ON master/replicant:

# 1. create replication user 
# event better IP-Range instead of % -> 192.168.56.% 
CREATE USER repl_multi@'%' identified by 'your_secret_pass'
GRANT REPLICATION SLAVE ON *.* TO 'repl_multi'@'%'

# 2. test connection with that user 
# in our case on same server 
# explicitly host, because that's how we use it in CHANGE MASTER 
mysql -urepl_multi -p -h127.0.0.1 

# 3. Daten auf master ausspielen und master-data notieren 
# CHANGE MASTER steht relativ am Anfang der Datei 
mysqldump --all-databases --single-transaction --source-data=2 --routines --events --flush-logs --delete-source-logs -uroot -p > all-databases.sql

-> ON slave/replica:

# 1. be sure, that server does not have same server_id // server uuid 

# --> Delete auto.cnf in datadir 

# -> change server_id in my.cnf in [mysqld] section to > 1 
# must be unique across all servers in master/slave replications network 
# e.g. 
server_id = 2 

# 2. Restart server (in our case mysql2 is the replica) 
net stop mysql2
net start mysql2

# 3. Import data into slave/replica from master 
# port of our replica is 3308 
mysql -uroot -p --port=3308 -h 127.0.0.1 < all-databases.sql 

# 4. Construct change master -> sql command 
# with master_pos, master_log_file from dump 
# CHANGE MASTER is the same as CHANGE REPLICATION SOURCE 
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='127.0.0.1',
SOURCE_USER='repl_multi',
SOURCE_PASSWORD='password',
SOURCE_LOG_FILE='binlog.000026',
SOURCE_LOG_POS=156
FOR CHANNEL 'replicant-1';

# 5. start replica 
START REPLICA FOR CHANNEL 'replicant-1'

# 6. Check on slave if you succeeded
show replica status; 
# or 
show slave status; 

# look for slave_io_running -> YES  

# look for slave_sql_running -> YES 

# If not look for errors within the output 

```
## Walkthrough (replicant-2) 

```
1. neues Verzeicnnis erstellen
2. richtige Rechte geben. -> NETWORK SERVICES
Trainer08 IT-Schulungen11:19
3. my.ini kopieren aus bisheriger Instanz 
4. my.ini anpassen: port, datadir, server-id
5. cmd.exe als Admin ausführen 
6. in das bin -verzeichnis wechseln 
cd C:\Program Files\MySQL\MySQL Server 8.0\bin
7. Datenverzeichnis initial erstellen
mysqld --defaults-file=C:\ProgramData\MySQL\MYSQL-basic\my.ini --initialize --console
8. password kopieren / temporäres
IXahAsElk4)V
9. Service installieren
10. mit Server verbinden (konsole) und password setzen
mysql -uroot -p --port=3310
mysql> alter user root@localhost identified by 'password'
11. show master status;
Daten notieren. 
12. Replications-User einrichten 
create user multi@'%' identified by 'password';
grant slave replication on *.* to multi@'%';
13. Test connection with multi user
show grants

14. On slave 
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='127.0.0.1',
SOURCE_USER='multi',
SOURCE_PORT=3310,
SOURCE_PASSWORD='password',
SOURCE_LOG_FILE='binlog.000002',
SOURCE_LOG_POS=10403
FOR CHANNEL 'replicant-2';

CHANGE REPLICATION FILTER REPLICATE_DO_DB = (dauerversuche) FOR CHANNEL 'replicant-2'START REPLICA FOR CHANNEL 'replicant-2';
START REPLICA FOR CHANNEL 'replicant-2';
SHOW REPLICA STATUS;
```


## Show the state of replication in performance schema 

```
# all slaves 
show slave status;

# specific slave
show slave status for channel 'replicant1';

# more information in performance_schema 
use performance_schema;
select * from performance_schema.replication_connection_status \G
```

