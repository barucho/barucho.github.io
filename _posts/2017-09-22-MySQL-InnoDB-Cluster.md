---
layout: post
title: "MySQL InnoDB Cluster"
date: 2017-09-22
---

This Post is about MySQL InnoDB Cluser  (not MySQL Cluser NDB)

MySQL InnoDB Cluser is based on MySQL group replication and by Joining at least three servers ensures a high availability cluster ,With 

So How its works ? 





MySQL InnoDB cluster is a collection of products that work together to provide a complete High Availability solution for MySQL. A group of MySQL servers can be configured to create a cluster using MySQL Shell. In the default single-primary mode, the cluster of servers has a single read-write primary. Multiple secondary servers are replicas of the primary. Creating a cluster with at least three servers ensures a high availability cluster. A client application is connected to the primary via MySQL Router. If the primary fails, a secondary is automatically promoted to the role of primary, and MySQL Router routes requests to the new primary. Advanced users can also configure a cluster to have multiple-primaries.










***have fun***