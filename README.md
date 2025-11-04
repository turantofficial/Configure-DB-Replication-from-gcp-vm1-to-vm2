# Configure-DB-Replication-from-gcp-vm1-to-vm2

Configure DB-Replication master > slave

Step by Step:     
Deactivate os-login in InstanceA + InstanceB
 
InstanceA **Master**:
```
mariadb-dump -u tosun -pSEC --no-data --databases db1 db2 db3 | pv | ssh user@10.10.10.10 "cat > /mnt/db3-gcp-databases/InstanceA_schema.sql"

mariadb-dump -u tosun -pSEC --skip-lock-tables --single-transaction --flush-logs --master-data=2 --databases db1 db2 db3 | pv | ssh user@10.10.10.10 "cat > /mnt/db3-gcp-databases/mysqldumpInstanceA.sql"
```


InstanceB **Slave**:
```
chown mysql:mysql /mnt/db3-gcp-databases/InstanceA_schema.sql
chown mysql:mysql /mnt/db3-gcp-databases/mysqldumpInstanceA.sql

pv /mnt/db3-gcp-databases/InstanceA_schema.sql | mariadb
pv /mnt/db3-gcp-databases/mysqldumpInstanceA.sql | mariadb

head -n 50 /mnt/db3-gcp-databases/mysqldumpInstanceA.sql | grep -i "CHANGE MASTER"
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.001', MASTER_LOG_POS=103;

STOP SLAVE 'replic_jvapps_gcp_db3';

CHANGE MASTER 'replic_InstanceA' TO
    MASTER_HOST='10.10.10.10',
    MASTER_USER='replic_InstanceA',
    MASTER_PASSWORD='SEC',
    MASTER_LOG_FILE='mysql-bin.001',
    MASTER_LOG_POS=103
    MASTER_SSL=0;

set global replic_jvapps_gcp_db3.replicate_do_db = 'db1,db2,db3';
set global replic_jvapps_gcp_db3.replicate_ignore_db = 'db4,db5,db6';


START SLAVE 'replic_InstanceA';

show all slaves status\G;
```
