## Replication

Replication is necessary for all database systems because failures happen.

MongoDB supports two types of Replication:
-  _Master Slave Replication_
-  _Replica Sets_

`Master-Slave` replication and `Replicasets` both use same replication mechanism i.e A single primary node receives all the writes, and all the secondary nodes read and apply those writes to themselves asynchronously.

Replicaset mechanism has some advantages over Simple Master-Slave model. i.e.
- _Automated Failover_
- _Easier Recovery_

Due to this it is advisable to use **Replicaset model** in production.

In addition, to providing `redundancy` and `failover protection`, replication simplifies maintenance tasks as well by providing ability to run expensive operations on secondary nodes.

> Since building indexes are expensive we can first build them on secondary, swap the secondary with a primary and then build again on the new secondary.

## ReplicaSets Replication Model

The minimum recommended replicaset configuration consists of **3 nodes**.
  
Two of these 3 nodes, serve as the persistent **mongod instances**. Either of these 2 nodes can act as the `primary` but both will have the full copy of data.
   
Third node will act as the **Arbiter**. An arbiter node does not contains the copy of data and only acts as on observer. During the time of failures, arbiter node helps to elect the new primary.

[Fig: chapter MongoDBreplication 1(a)]

## How Replication Works

Replicasets rely on two basic mechanisms: 
- _Oplog_
- _HeartBeat_

At the heart of Mongodb replication stands the **`Oplog`**.

The **Oplog** is a _capped collection_ that lives in a database called **local** on every node that contains replication data.

Every time a client writes to primary, an entry with enough information to reproduce the write is automatically added to _primary's oplog_.

Once this write is replicated to secondary server, that secondary server also stores the record of write in its _local oplog_.
Each **Oplog** entry has a _BSON timestamp_, and all the secondaries use this timestamp to keep track of the writes they have applied.

#### The Local Database
The **`Local`** database stores all the _metadata_ related to the replicaset and the _Oplog_ data. 

This **Local** db is never replicated and is local to the server it resides on. Thus, its called _local_ DB.

Following are the collections present inside the Local database:
- _`me`_, _`slaves`_ : Used to implement write concern.
- _`system.indexes`_: contains all index info.
- _`system.replset`_: stores replicaset config.
- _`oplog.rs`_: stores the oplog.
- _`replset.minvalid`_: contains information about initial replicaset sync.
