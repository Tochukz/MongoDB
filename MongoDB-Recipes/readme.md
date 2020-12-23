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
Use the `deleteOne()`, `deleteMany()` or `remove()` method to delete a document from a collection.   
```
> db.users.deleteOne({username: 'John98'})
> db.users.remove({username: 'Macus55'})
> db.users.deleteMany({id: {$gt: 7}})
```
To delete all the documents in a collection use the `deleteMany()` method with `{}` argument.  
```
> db.user.deleteMany({})
```   

__MongoDN Import and Export__  
We can use the import tool to import data from `JSON`, `CSV` and `TSV` files.   
We can also use the export tool to export data in `JSON` or `CSV` format.   

Import data from a `.csv` files using `mongoimport`
```
> mongoimport --db recipes --collection users --type csv --headerline --file D:\workspace\MongoDB\MongoDB-Recipes\chp2\users.csv
```  
Export a data from a collection to a `.json` file  
```
> mongoexport --db recipes --collection users --out D:\workspace\MongoDB\MongoDB-Recipes\chp2\users.json
```

__Embedded Documents in MongoDB__  
Consider the document `{id: 1001, name: "John", marks: {english: 59, maths: 67}, result:"pass"}`. The `marks` field contains an embedded document.  
To find a document with a matching embedded or nested document:  
```
> db.student.find({marks: {math: 67, eng: 59}})
```
This should return the example document above.   
__NB:__ The embedded document must match for each key-value pair.   
&nbsp; &nbsp; &nbsp; &nbsp; Doing `db.student.find({marks: {math: 67}})` will not return the same result because the `eng: 59` key value pair is missing.   

To query the nested document without matching its entire key-value pair, do:
```
> db.student.find({"marks.math": 67})
```
This will return all document whose `marks` nested document has a `math` score of 67 irrespective of their english score.

__Working with Arrays__  
Insert many documents containg with an array field called `lang`  
```
> db.devs.insertMany([{name: 'Chucks', langs: ['PHP', 'JavaScript', 'C#']}, {name: 'Chike', langs: ['PHP', 'Java', 'Python']}])
```

To query for a document with a matching array field:  
```
> db.devs.find({langs: ['PHP', 'JavaScript', 'C#']})
```
Note that the order of elements in the array for the matching field must be the same.   

To match an array having at least one particular element value, do:  
```
> db.devs.find({langs: 'PHP'})
```
Lets update the collection with a new array called scores:   
```
> db.devs.updateOne({name: 'Chucks'}, {$set: {scores: [67, 82, 93] } })
> db.devs.updateOne({name: 'Chike'}, {$set: {scores: [57, 72, 59] } })
```
To find the documents whose `scores` array contains at least one value that is greather than 80
```
> db.devs.find({scores: {$gt: 80}})
```  
You can use a compound filter condition to obtain a range of value:
```
> db.devs.find({scores: {$gt: 60, $lt: 70}})
```  


__Restricting the Fiels Returned from a query__  
Lets create an example collection to use  
```
> db.studentdetails.insertMany([
  {
    name: "John", result: "pass", marks: { english: 25, maths: 23, science: 25}, grade: [{ class: "A", total: 73}]
  },
  {
    name: "James", result: "pass", marks: { english: 24, maths: 25, science: 25}, grade: [{ class: "A", total: 74}]
  },
  {
    name: "James", result: "fail", marks: { english: 12, maths: 13, science: 15}, grade: [{ class: "C", total: 40}]
  }
])
```
If we want to get only specified fields from a query, we pass a second object argument to the `find()` method with the desired fields as keys and a value of `1`.  
```
> db.studentsdetail.find({name: 'John'}, {name: 1, result: 1})
```  
This query will return the `name`, `result` and `_id` field of all document that matches `{name: 'John'}`.  

If you don't want to `_id` field you can explicitly set it to `0`.   
```
> db.studentsdetail.find({name: 'John'}, {name: 1, result: 1, _id: 0})
```  
To select specific fields from an embedded document, use the dot notation  
```
> db.studentdetails.find({}, {name: 1, result: 1, "marks.science": 1})
```  
The returns the `name`, `result`, `science` and `id` field of all the documents in the collections.  The `science` field is in an embedded document whose key is `marks`.  

__Querying Null or Mising Fields__  
Let create an example collection:  
```
> db.users.insertMany([{name: 'Mark', city: 'Cape Town'}, {name: 'John', city: null}, {name: 'Smith'}])
```
To query for the documents that have `city` field with value of `null` or does not have `city` field at all  
```
> db.users.find({city: null})
```
This will return the last two documents show in the collection above.   
To return only those document with `city` value of `null` and NOT those that have not `city` field  
```
> db.users.find({city: {$type: 10}})
```
This will return onlt the second document and not the third.  

__Existence Check__  
Use the `$exists` operator to check if a specific field exits in a collection  
```
> db.users.find({city : {$exists: true}})
```  
This will return only those document that have a `city` key.

__Iterate a Cursor__  
The _db.collection.find()_ returns a cursor that automatically iterates the first 20 documents by default.   
We can asing the cursor to a variable and manually iterate the documents.  
```
> var myCursor = db.studentdetails.find();
> while(myCursor.hasNext()) {
    print(tojson(myCursor.next()))
  }
```  
Alternatively, we can use `forEach()` method to iterate the cursor.  
```
> var myCursor = db.studentdetails.find();
> myCursor.forEach(printjson)
```  

__working with the limit() and skip() methods__  
```
> db.studentdetails.find({name: 'John'}).limit(1)
> db.studentdetails.find().skip(25)
> db.studentdetails.find({name: 'John'}).skip(5).limit(2)
```

__Working with Node.js and MongoDB__  

## Chapter 3: Data Modeling and Aggregation
There are two data model design:
* __Embedded data models:__ It uses embedded documents.
* __Normalized data models:__ It describes relationships using document references.

__Query Document References__   
Lets create two example collections:   
```
> db.publishers.insert({ id:"Apress", name: "Apress", founded: 1999, location:"US"})
> db.authors.insertMany([
      {
        id: 123456,
        title: "Practical Apache Spark",
        author:["Subhashini Chellappan", "Dharanitharan Ganesan"],
        published_date: ISODate("2018-11-30"),
        pages: 300,
        language: "English",
        publisher_id: "Apress"
    },
    {
      id: 456789,
      title: "MongoDB Recipes",
      author: [ "Subhashini Chellappan"],
      published_date: ISODate("2018-11-30"),
      pages: 120,
      language: "English",
      publisher_id: "Apress"
    }
])
```
Now lets do a left outer join using the `$lookup` operator.
```
> db.publishers.aggregate([{$lookup: {from: "authors", localField: "id", foreignField: "publisher_id", as: "authors_docs"}}]).pretty()
```
This will return a collection of `publishers` documents and embedded all their corresponding author documents as an array under the `author_docs` key.  

__Model Tree Structures with Parent Reference__   
Lets modify and insert some documents:
```
> db.student.updateOne({name: "John"}, {$set: {parent: "Kelvin"}})
> db.student.updateOne({name: "Jane"}, {$set: {parent: "Kelvin"}})
> db.student.insertOne({_id: "Kelvin", parent: null})
```
We can retrieve the parent document of a given document as follows
```
> db.student.findOne({name: 'John'}).parent
```  
And to retrieve the immediate children of the parent
```
> db.student.find({parent: 'Kelvin'})
```

__Tree Structure with Child reference__  
Let update a documents and add child referenced to it  
```
db.student.updateOne({name: "John"}, {$set: {children: []}})
db.student.updateOne({name: "Jane"}, {$set: {children: []}})
db.student.updateOne({_id: "Kelvin"}, {$set: {children: [ObjectId("5eb3bf0ed528d2a5d46929ab"), ObjectId("5eb3bf0ed528d2a5d46929ac")]}})
```  
Note that the elements of the children array are the `_id`s of the child documents.  
To get all the children of a document.  
```
> db.student.find({_id: "Kelvin"}).children
```  
To find all parent documents that have a given child:  
```
> db.student.find({children: ObjectId("5eb3bf0ed528d2a5d46929ab")})  
```  

__Tree Structure with Array of Ancestors__  
Let insert a document having an array of ancestors  
```
> db.author.insert( { _id: "Practical Apache Spark", ancestors: ["Subhashini", "Books" ], parent: "Books" } )
```
The elements of the `ancestors` array must be the `_id`s of the ancestors.   
To get all the ancestors of a document
```
> db.author.findOne( { _id: "Practical Apache Spark" } ).ancestors
```  
And to find al the descendants that have a given ancestor  
```
> db.author.find( { ancestors: "Subhashini" } )
```

__Aggregation__  
MongoDB provides the following aggregation operations:  
* Aggregation pipeline  
* Map-reduce function  
* Single-purpose aggregation methods  

__Aggregation Pipeline__  
Lets insert a many documents to use for aggregation  
```
> db.orders.insertMany([
    {custID: "10001",amount: 500,status: "A"},
    {custID: "10001",amount: 250,status: "A"},
    {custID: "10002",amount: 200,status: "A"},
    {custID: "10001",amount: 300, status: "D"}
]);
```
Use the `aggregate()` function  
Lets group the collection by `custID` and sum-up their amounts  
```
> db.orders.aggregate({$group: {_id: "$custID", TotalAmount: {$sum: "$amount"}}});
```
Let carry out the same aggregation but this time filter out documents whose `status`  is not "A".  
```
> db.orders.aggregate({$match: {status: "A"}}, {$group: {_id: "$custID", TotalAmount: {$sum: "$amount"}}});
```  
Use the `$avg` operator to calculate the average   
```
> db.orders.aggregate({$group: {_id: "$custID", AverageAmount: {$avg: "$amount"}}});
```    

__Map-Reduce__   




## Chapter 8: MongoDB Security  
MongoDB supports the following authentication mechanism:  
* _SCRAM_ The default, it uses username and password.  
* _x.509 certificates_   

MongoDB Enterprise Edition also supports _Lightweight Directory Access Protocol_ (LDAP) proxy authentication and _Kerberis_ authentication.  

__Roles__   
The following are MongoDB built-in roles:   
* DB User Roles
* Cluster Admin Roles  
* DB Admin Roles
* Super User Roles
* All Databases Roles  

__Roles and Previleges__  

| Role              | Previleges         |
|-------------------|--------------------|
| User Roles        | read, readwrite    |  
|                   |                    |  

__Create a Superuser__  
```
> user myDB
> db.createUser({user: "username", pwd: "pass", roles: [{role: "userAdminAnyDatabase", db: "myDB"}]})
```
To continue from here
