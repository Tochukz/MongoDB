# MongoDB Recipes (2020)
__By Subhashini Chellappan and Dharanitharan Ganesan__  

_Replication_ is the process of creating and managing duplicate versions of a database across servers to provide redundancy and increase availability of data. _Sharding_ distributes data across servers to increase availability.  

## Chapter 1: MongoDB Features and Installation
NoSQL databases are _polygot persistent_, meaning there are different ways to store the data based on the requirements.  

__CAP Theorem__  
The _CAP_ theorem states that any distributed system can satisfy only two of these three properties:
* _Consistency_ - every read fetches the last write
* _Availability_ - read and write always succeed
* _Partition tolerance_ - the system will continue to function even when there is a data loss or system failure.
The partition tolerance property is a must for NoSQL databases.   

__BASE Approach__  
NoSQL databases are based on the _BASE_ approach which stands for:
* __B__asic __A__vailability - The database should be available most of the time
* __S__oft state - Temporal inconsistency is allowed  
* __E__ventual consistency - They System will come to a consistent state after a certain period.  

__Types of NoSQL Databases__  

| Data Model       | Example            |
|------------------|--------------------|
|Key/Value Store   | Dynamo DB, Riak    |
|Column Store      | HBase, Big Table   |
|Document Store    | MongoDB, CouchDB   |  
|Graph Database    | Neo4j              |

__MongoDB Uses BSON__  
MongoDB represents JSON documents in a binary-encoded format, _Binary JSON (BSON)_, behind the scenes. BSON extends the JSON model to provide additional data types such as dates that are not supported by JSON.  

__Replication__  
Replication is a process of copying an instance of a database to a different database server to provide redundancy and high availability. Replication provides fault tolerant against the loss of a single database server.  In MongoDB, the replica set is a group of _mongod_ processes that maintain the same data set.  

__Sharding__
MongoDB uses _Sharding_ to support large data sets and high-throughput operations. _Sharding_ is a method for distributing data across multiple systems.

_JOINs_ and _GROUP_BY_ in RDBMS is synonymous to _Embedded document_ and _Aggregation pipeline_ respectively in MongoDB.   

__ACID Transactions__  
_ACID_ transactions obey the following principles:  
* __A__tomicity - Either the entire transaction takes place or nothing
* __C__onsistency - Consistency must be maintained before and after the transaction  
* __I__solation - Multiple transactions occur independently without interference.  
* __D__urability - The changes of a successful transaction occurs even system failure occurs.  

__Working with Database Commands__  
Create a Database  
```
> use RecipeDB
```  

Show the current Database  
```
> db
```  

Drop a Database  
```
> use RecipeDB  
> db.dropDatabase()
```
NB: If no database is select, using the _use_ statement, the default database will be dropped.   
List all databases  
```
> show dbs
> show Databases
```
Any of the two command will do.  

Display the MongoDB version  
```
> db.version()
```  

Display help information  
```
> db.help()
```

## Chapter 2: MongoDB CRUD Operations
__Collections__  
You can create a collection by simple referining to it:
```
> db.people.insert({name: 'Chucks', City: 'Cape Town'});
```
This will create the `people` collection if it does not already exists and insert the document in the collection.   
Alternatively you can create a collection explicitely by using the `createCollection` method:
```
> db.createCollection('people')
```
We can see all existing collections by doing
```
> show collections
```
__Capped Collection__   
Capped colletions have fixed sizes, and they work like a circular buffer: Once filled, they overwrite the oldest document to make room when new onces are added.  Their limitations are
* you cannot shard them  
* you cannot use aggregation pipeine operator $out
* You cannot delete their documents  
To create a capped collection:  

```
> db.createCollection('students', {capped: true, size: 1000, max: 2})
```  
`size` is the maximum size of the collection in bytes, `max` is the maximum number of documents allowed.    
To check if a collection is capped:  
```
> db.students.isCapped()
```  
To retrive documents of a collection in reverse order of  insertion use the `sort` method with the `$natural` parameter:  
```
db.students.find().sort({$natural: -1})
```  

__Create Operations__  
There are three methods used to insert a document into a collection: `insertOne()`, `insetMany()`, `insert()`.    
The `insert` method may be used to inset a single document or an array of documents.  
```
> db.users.insertOne({username: 'John98'})
> db.users.insertMany([{username: 'Yusuf001'}, {username: 'Macus55'}])
```
The `insertOne` operation will return an object with `insertedId` property whose value is the geenrated `_id` of the newly created method. Similarly, the `insertMany` method returns an object with a `insertIds` property whose value is an array of the `_id`s of the newly created documents.  

__Read Operations__  
Use the `find` method to query a collection.  
Get all the documents in a collection  
```
> db.users.find()
```
Get all document was `name` key has a value of `John98`  
```
 db.user.find({ name: 'John98' })
```  
_NB:_ that the value of a key is case sentitive.

Get all users whoe name is `John` and 


Comparison Operators

| Operator | Meaning                  |
|----------|--------------------------|
| $gt      | greater than             |
| $lt      | less than                |
| $gte     | greater than or equal to |
| $lte     | less than or equal to    |
| $eq      | equal to                 |

Show all users whose age are than 18 and above  
```
> db.user.find({age: {$gte: 18}})
```

Show all users who are 18 or older _AND_ lives in Cape Towm  
```
> db.users.find({ city: "Cape Town", age: { $gte: 18 } })
```  

Use the `$or` operator to select users who live in Cape Town _OR_ Joburg.  
```
> db.users.find({ $or: [{ city: 'Cape Town'}, { city: 'Joburg' }] })
```

__Update Operations__  
You can use one of four method to modify a document:  
`updateOne()`, `updateMany()`, `replaceOne` and `update()`  Let create a new collection to play with  
```
> db.students.insertMany([{id: 1, name: "John", marks: {math: 67, eng: 57}, result: "pass"}, {id: 2, name: "Jane", marks: {math: 49, eng: 86}, result: "fail"}]) 
```

Use the `$set` operator to update the second document  
```
> db.students.updateOne({ id: 2 }, {$set: { "marks.math": 51, result: 'pass' }})
```

Use the `updateMany()` method to update all student marks who passed  
```
> db.student.updateMany({ result: 'pass' }, {$set: { "marks.math": 50, "marks.eng": 50 }})
```   
Use the `replaceOne()` method to completely replace the document that march the criteria of the first argument.   
```
> db.students.replaceOne({id: 1}, {name: "John", result: 'fail'})
```  
You will notice that the first document no longer have the `id` and `marks` field. This is because the replacement document(i.e second argument) did not define these fields.  The `update()` method behaves in the same way as the `replaceOne()` method.  

__Delete Operations__  
