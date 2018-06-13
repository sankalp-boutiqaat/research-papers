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

#### How Secondaries Replicate write ?

Each secondary node has its own local oplog.




##Deployment
1. First Bootup 3 mongod instance:

mongod --replSet "rs0" --port=12323 --dbpath=/data/replica/rs0/12323
mongod --replSet "rs0" --port=12424 --dbpath=/data/replica/rs0/12424
mongod --replSet "rs0" --port=12525 --dbpath=/data/replica/rs0/12525

2. Then from the command line run 'mongo --port-12323' and run below command.

rs.initiate( {
   _id : "rs0",
   members: [
      { _id: 0, host: "localhost:12323" },
      { _id: 1, host: "localhost:12424" },
      { _id: 2, host: "localhost:12525" }
   ]
})

Now, the replicaset is configured.

rs.conf() : Displays replicaset config object.
rs.status(): Displays replicaset status. This can be used to identify primary node.

NOTE: In order to read from slave node, we need to execute command "rs.slaveOk()". This command is only valid for a single session.
Thus each new session if wants to read from slave need to execute this first.


Connecting to replicaset from mongoshell:

we can connect to a replicaset using below command:
mongo --host rs0/localhost:12424,localhost:12525  (no need to specify all the nodes)

the above statement translates to:
mongodb://localhost:12424,localhost:12525/?replicaSet=rs0