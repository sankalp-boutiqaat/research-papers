## Sharding

A MongoDB Sharded Cluster consists of 3 main components:

1. Shard
- Each shard contains a subset of the sharded data. Each shard can be deployed as a replica set.

2. Mongos
- The mongos acts as a query router, providing an interface between client applications and the sharded cluster.

3. Config Servers
- Config servers store metadata and configuration settings for the cluster. As of MongoDB 3.4, config servers must be deployed as a replica set (CSRS).

> MongoDB shards data at the collection level i.e distributes the collection data across the shards in the cluster. 

### Shard Key:

Mongodb partitions the collection data using shard key. The shard key consists of an immutable field (i.e a field that does not changes over time) or combination of fields.

A Shard key is chosen at the time of sharding the collection. Once chosen and the collection is sharded you cannot change it.

> NOTE: A sharded collection can have only one shard key. 

Your collection must have an index that starts with the shard key.

### Chunks:

Now the collection that is to be sharded is divided into small small sets of data, where each such set is called a chunk.

These chunks are migrated from one shard to another by the help of Shard chunk Balancer. 

### Considerations before sharding:

You cannot change the shard key after sharding, nor can you unshard a sharded collection.
If queries do not include the shard key or the prefix of a compound shard key mongos performs a broadcast operation, querying all shards in the sharded cluster.
These scatter/gather queries can be long running operations.

### Sharded and non sharded collections:

A database can have a mixture of sharded and non-sharded cluster.

Sharded collections are partitioned and distributed across all shards of the cluster. 
Unsharded collections are stored in a primary shard. (Each database has its own primary shard).

### Connecting to a sharded cluster.

In a sharded cluster, we dont connect with any specific shard we connect with the mongos router to interact with any collection.

### Sharding Strategies:
@todo: pending

### FAQ's

- How do I get list of databases that have sharding enabled?
```
use config
db.databases.find( { "partitioned": true } )
```

- How do I get Status of sharded cluster?
```
sh.status()
```

- How do I get information about mongos servers?
```
use config
db.mongos.find().pretty()
```

- How do I get information about config  servers?
```
Login into one of the config servers and run rs.status()
```

- How do I get information about shards in cluster?
```
use config
db.shards.find().pretty()
```

- How do I get primary shard for each database?
```
use config
db.databases.find()
```

- How do I get information about each shards replicaset?
 Login into the shard and run 
 rs.status()
