## Replication

Replication is necessary for all database systems because failures happen.

MongoDB supports two types of Replication:
-  Master Slave Replication
-  Replica Sets

Master-Slave replication and Replicasets both use same replication mechanism i.e A single primary node receives all the writes, and all the secondary nodes read and apply those writes to themselves asynchronously.

Replicaset mechanism has some advantages over Simple Master-Slave model. i.e.
- Automated Failover
- Easier Recovery

Due to this it is advisable to use Replicaset model in production.

In addition, to providing `redundancy` and `failover protection`, replication simplifies maintenance tasks as well by providing ability to run expensive operations on secondary nodes.

> Since building indexes are expensive we can first build them on secondary, swap the secondary with a primary and then build again on the new secondary.

## ReplicaSets Model

The minimum recommended replicaset configuration consists of **3 nodes**.
  
Two of these 3 nodes, serve as the persistent **mongod instances**. Either of these 2 nodes can act as the `primary` but both will have the full copy of data.
   
Third node will act as the arbiter. An arbiter node does not contains the copy of data and only acts as on observer. During the time of failures, arbiter node helps to elect the new primary.

[Fig: chapter MongoDBreplication 1(a)]

## How Replication Works

Replicasets rely on two basic mechanisms: 
- Oplog
- HeartBeat

At the heart of Mongodb replication stands the `Oplog`. The Oplog is a capped collection that lives in a database called 'local' on every node that contains replication data.
Every time a client writes to primary, an entry with enough information to reproduce the write is 
automatically added to primary's oplog.
Once this write is replicated to secondary server, that secondary server also stores the record of write in its local oplog.
Each Oplog entry has a BSON timestamp, and all the secondaries use this timestamp to keep track of the writes they have applied.

#### The Local Database
The `Local` database stores all the metadata related to the replicaset and the Oplog data. This `Local` db is never replicated and is local to the server it resides on. Thus its called `local` DB.
Following are the collections present inside the Local database:
- me, slaves : Used to implement write concern.
- system.indexes: contains all index info.
- system.replset: stores replicaset config.
- oplog.rs: stores the oplog.
- replset.minvalid: contains information about initial replicaset sync.


