---
layout: post
title: "MySQL InnoDB Cluster"
date: 2017-09-22
---

This Post is about MySQL InnoDB Cluser  (not MySQL Cluser NDB)

MySQL InnoDB Cluser is based on MySQL group replication and by Joining at least three servers ensures a high availability cluster ,With 

So How its works ? 





MySQL InnoDB cluster is a collection of products that work together to provide a complete High Availability solution for MySQL. A group of MySQL servers can be configured to create a cluster using MySQL Shell. In the default single-primary mode, the cluster of servers has a single read-write primary. Multiple secondary servers are replicas of the primary. Creating a cluster with at least three servers ensures a high availability cluster. A client application is connected to the primary via MySQL Router. If the primary fails, a secondary is automatically promoted to the role of primary, and MySQL Router routes requests to the new primary. Advanced users can also configure a cluster to have multiple-primaries.



**befor we bagin** 
* use MySQL 5.7.19+

* we will need the group_replication plugin to see if you the the plugin installed : 

```sql
SHOW PLUGINS;
| group_replication          | ACTIVE   | GROUP REPLICATION  | group_replication.so | GPL     |

```
* to install 

```sql
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
```





## lets configure 3 nodes one master ... 
i will created the 3 nodes on the same machine by using diffrent data dir and port numbers 


### first create directories 

```bash 
mkdir -p /u01/data/mysql_group1/{1,2,3}
mkdir -p /u01/etc/{1,2,3}
``` 
### initialize mysql metadata  
the *--initialize-insecure* flage will create root user witout passowrd.

```bash 
mysqld --initialize-insecure  --datadir=/u01/data/mysql_group1/1
mysqld --initialize-insecure  --datadir=/u01/data/mysql_group1/2
mysqld --initialize-insecure  --datadir=/u01/data/mysql_group1/3
```


### the my.cnf for all 3 nodes 
create the relevnt config file in /u01/etc/{1,2,3} 

```conf
####mysql1
[mysqld]

# server configuration
datadir=/u01/data/mysql_group1/1

port=24801
socket=/u01/data/mysql_group1/1/s1.sock
## REPLICATION
server_id=1
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
log_bin=binlog
binlog_format=ROW
## group REPLICATION
transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
loose-group_replication_start_on_boot=off
loose-group_replication_local_address= "127.0.0.1:24901"
loose-group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
loose-group_replication_bootstrap_group= off
loose-group_replication_single_primary_mode=FALSE
loose-group_replication_enforce_update_everywhere_checks= TRUE

```
```conf
######mysql2
[mysqld]

# server configuration
datadir=/u01/data/mysql_group1/2

port=24802
socket=/u01/data/mysql_group1/2/s2.sock
## REPLICATION
server_id=2
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
log_bin=binlog
binlog_format=ROW
## group REPLICATION
transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
loose-group_replication_start_on_boot=off
loose-group_replication_local_address= "127.0.0.1:24902"
loose-group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
loose-group_replication_bootstrap_group= off
loose-group_replication_single_primary_mode=FALSE
loose-group_replication_enforce_update_everywhere_checks= TRUE


```
```conf
####mysql3
[mysqld]

# server configuration
datadir=/u01/data/mysql_group1/3

port=24803
socket=/u01/data/mysql_group1/3/s3.sock
## REPLICATION
server_id=3
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
log_bin=binlog
binlog_format=ROW
## group REPLICATION
transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
loose-group_replication_start_on_boot=off
loose-group_replication_local_address= "127.0.0.1:24903"
loose-group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
loose-group_replication_bootstrap_group= off
loose-group_replication_single_primary_mode=FALSE
loose-group_replication_enforce_update_everywhere_checks= TRUE
```


###start node 1 
```bash 
mysqld --defaults-file=/u01/data/mysql_group1/1/my.cnf
```
###create replication user 

```bash
mysql -u root -p --port 24801 --protocol=tcp

```

```sql
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%' IDENTIFIED BY 'rpl_pass';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='rpl_pass' FOR CHANNEL 'group_replication_recovery';
```

### bootstrap the group replication: 
* this need to be run only! on the first node!!

```bash
mysql -u root -p --port 24801 --protocol=tcp

```

```sql
SET GLOBAL group_replication_bootstrap_group=ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group=OFF;
```

```sql
SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 1ce52f01-e615-11e6-9335-080027ad34e3 | host72      |       24801 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
1 row in set (0.00 sec)

```


***have fun***