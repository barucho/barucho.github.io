---
layout: post
title: "PostgreSQL replication On Docker"
date: 2017-09-19
---

I created a Postgresql docker that can automatically start as slave
on the docker docker-entrypoint.sh i added script that 

1. chack if the server/Docker is SLAVE 
2. recover from master 
3. replace config file 
4. create recover file 

To run it use this code [PG_Docker_replication](https://github.com/barucho/PG_Docker_replication.git) 


### to build :
```bash 
docker build -t postrep .
```
### to clean:
```bash 
docker rmi $(docker images | grep "^<none>" | awk '{print $3}')
```
### to run  :
```bash 
docker run -d --name master  postrep
docker run -d --name slave -e SLAVE=<master_ip> postrep
```
### run failover:
```bash 
docker exec -it -u postgres slave  bash -c 'pg_ctl promote'
```
### show log:
```bash
docker logs slave
```
### show replication status:
```bash 
docker exec -it -u postgres master  bash -c 'echo "select * from pg_stat_replication;" |psql' 
docker exec -it -u postgres slave  bash -c 'echo "select now()-pg_last_xact_replay_timestamp();" |psql'
```


have fun