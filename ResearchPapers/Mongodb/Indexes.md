## Indexes
---

Indexes are necessary, without proper indexing mongoDB needs to scan whole collection in order to prepare data.

### Single Field Indexes

An Index that contains only one field is called "Single Field Index". A Single field index can be ascending or descending.
```json
//creates an ascending index on field score.
db.records.createIndex( { score: 1 } )

//Creates a descending index on field score.
db.records.createIndex( { score: -1 } )
```
#### Creating Index on Embedded field.
```json
db.records.createIndex( { "location.state": 1 } )
```
The above index will support following queries.
```
db.records.find( { "location.state": "CA" } )
db.records.find( { "location.city": "Albany", "location.state": "NY" } )
```
NOTE: The direction of the index i.e Ascending or Descending does not matter in case of single field indexes. As it can be looked up from both sides forward as well as reverse.

### Compound Index
An index that contains multiple fields is called "Compound Field Index".
```json
//shows a compound index on item ans stock
db.products.createIndex( { "item": 1, "stock": 1 } )
```
The order in which the fields are specified in an index is important. This is because while creating index, mongo will put
all stock records inside of item records, so there will be no direct referencing to stock records.
[@todo: need a good representation for this.]

A compound index can support queries involving all the fields in index, and also queries involving index prefixes. 
```
// Below queries will be supported via the index we just built.
db.products.find( { item: "Banana" } ) //this one has a index prefix.
db.products.find( { item: "Banana", stock: { $gt: 5 } } )

//Below queries are not supported.
db.products.find( { stock: 5 } )
```
#### Sorting Compound Indexes

The above index that we created can be used by following queries for sorting result sets.
```json
db.products.find().sort( { "item": 1, "stock": 1 } )
db.products.find().sort( { "item": -1, "stock": -1 } )
```
However, None of the below queries will be able to use the index that we created for sorting results.
```json
db.products.find().sort( { "item": 1, "stock": -1 } )
db.products.find().sort( { "item": -1, "stock": 1 } )
```
This is becacuse of the structure in which indexes are stored in mongo.

[@todo definitely need to analyze this structure]

#### Index Prefixes

Index prefixes are the subset of whole index. Consider the following index:
```json
{ "item": 1, "location": 1, "stock": 1 }
```
Now, for this index following are the index prefixes:
```json
{ item: 1 }
{ item: 1, location: 1 }
```

Index prefixes helps us investigate, what all queries will be fulfilled by a given index.
All of the below queries can use the above index:
```json
db.products.find({item: "A"})
db.products.find({item: "A", location:"IN"})
db.products.find({item: "A", name:5})

```
None of the below queries can make use of index:
```json
db.products.find({localtion:"IN"})
db.products.find({localtion:"IN", stock: 4})
db.products.find({stock: 4})
```

### Index Intersection
[@todo: complete this]



### Building indexes on Background.
By default, creating an index on a collection blocks all operations on the database to which the collection belongs.
This is a huge pain as none of the collections within the database would take any Read/Write queries.
Also, any operation that requires a lock on all the databases will be blocked, such as: "listDatabases"

SO, there is an option to switch this index building task to background, so that there is no impact to other queries while index build is in process. 

Following query shows how to do it:
```json
db.people.createIndex( { zipcode: 1 }, { background: true } )
```

Note: Though the index creation operation is running in background, it will still block the mongo session from which you have issued the query.

Also, be aware that though index build is in backgroud it is still using a good amount of resources of server. So, database operations will be slower during this period.

### Building indexes on Replicasets and Sharded Clusters.
@todo:











