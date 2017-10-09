---
layout: post
title: "How To Install MyRocks"
date: 2017-10-09
---

MyRocks is a MySQL storage engine By Facebook that integrates with RocksDB. It provides improved flash storage performance through efficiencies in reading, writing, and storing data.


**To install :**

* first lets download the source.
```bash
git clone https://github.com/facebook/mysql-5.6.git
git submodule init
git submodule update
```
* cmake make file .. 
```bash
cmake . -DCMAKE_BUILD_TYPE=RelWithDebInfo -DWITH_SSL=system -DWITH_ZLIB=bundled -DMYSQL_MAINTAINER_MODE=0 -DENABLED_LOCAL_INFILE=1 -DENABLE_DTRACE=0 -DCMAKE_C
XX_FLAGS="-march=native"
```
* make the bin`s 
```bash
make -j2 
```
* install 
```bash 
make install
```
* create my.cnf 
```
[mysqld]
# server configuration
datadir=/u01/data/mysql/2
port=3306
socket=/u01/data/mysql/1/s1.sock
rocksdb
default-storage-engine=rocksdb
skip-innodb
default-tmp-storage-engine=MyISAM
log-bin
binlog-format=ROW
## REPLICATION
server_id=1
```

* create databases 
```bash 
mysqld --initialize-insecure  --datadir=/u01/data/mysql/1
```

* lets start the instance
```bash 
mysqld --defaults-file=/u01/data/mysql//etc/1/my.cnf &
```


***have fun***