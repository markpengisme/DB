# MongoDB

[manual](https://docs.mongodb.com/manual/)



## ENV

```sh
brew tap mongodb/brew
brew install mongodb-community


To have launchd start mongodb/brew/mongodb-community now and restart at login:
  brew services start mongodb/brew/mongodb-community
Or, if you don't want/need a background service you can just run:
  mongod --config /usr/local/etc/mongod.conf


brew services start mongodb/brew/mongodb-community
brew services list
```

- 安裝的東西包括
  - The mongod server
  - The mongos sharded cluster query router
  - The mongo shell

```sh
## 測試
mongo
mongotop
```

## Introduction

MongoDB 是文件導向資料庫，由 field/value 對組成，類似 JSON，文件儲存在集合(collection)裡，可以把它當成 dynamic schema table。

Documents 優點

- Documents correspond to native data types in many programming languages.
- Embedded documents and arrays reduce need for expensive joins.
- Dynamic schema supports fluent polymorphism.

MongoDB 特點：高性能、豐富的查詢語言、高可用性、水平擴展能力、支援多個儲存引擎

### Getting started

```js
// 顯示目前db 
db
// 使用 examples db, 沒有的話mongo會自動創造
use examples

// Insert(MongoDB adds an _id field with an ObjectId value if the field is not present in the document)
db.inventory.insertMany([
   { item: "journal", qty: 25, status: "A", size: { h: 14, w: 21, uom: "cm" }, tags: [ "blank", "red" ] },
   { item: "notebook", qty: 50, status: "A", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank" ] },
   { item: "paper", qty: 10, status: "D", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank", "plain" ] },
   { item: "planner", qty: 0, status: "D", size: { h: 22.85, w: 30, uom: "cm" }, tags: [ "blank", "red" ] },
   { item: "postcard", qty: 45, status: "A", size: { h: 10, w: 15.25, uom: "cm" }, tags: [ "blue" ] }
]);

// Select all
db.inventory.find({})
db.inventory.find({}).pretty()

// select filter
db.inventory.find( { status: "D" } );
db.inventory.find( { qty: 0 } );
db.inventory.find( { qty: 0, status: "D" } );
db.inventory.find( { "size.uom": "in" } );
db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } );
db.inventory.find( { tags: "red" } );
db.inventory.find( { tags: [ "red", "blank" ] } );

// select filter + specify field to retrun
// db.collection.find(<query document>, <projection document>)
db.inventory.find( {}, { item: 1, status: 1 } );
db.inventory.find( {}, { _id: 0, item: 1, status: 1 } );
```

### DB & Collcetions

```js
use myDB
// 隱式建立：當你第一次新增資料，會自動建立DB和Collcetions
db.myNewCollection1.insertOne( { x: 1 } )
db.myNewCollection2.insertOne( { x: 1 } )
db.myNewCollection3.createIndex( { y: 1 } )

// 顯式建立
db.createCollection() 
```

#### View

MongoDB view 是可查詢的物件，其內容由其他 collection 或 view上 的aggregation pipeline定義。  MongoDB不會將 view 內容儲存到硬碟，可以設定查詢權限。

[view 行為](https://docs.mongodb.com/manual/core/views/#behavior)

```js
// 法ㄧ
db.createCollection(
  "<viewName>",
  {
    "viewOn" : "<source>",
    "pipeline" : [<pipeline>],
    "collation" : { <collation> }
  }
)

// 法二
db.createView(
  "<viewName>",
  "<source>",
  [<pipeline>],
  {
    "collation" : { <collation> }
  }
)

// drop view
db.collection.drop() 

// modify view
collMod
```

#### [On-Demand Materialized Views](https://docs.mongodb.com/manual/core/materialized-views/)

Starting in version 4.2, MongoDB adds the `$merge`stage for the aggregation pipeline. This functionality allows users to create on-demand materialized views.

#### [Capped Collections](https://docs.mongodb.com/manual/core/capped-collections/)

Capped collections are fixed-size collections that support high-throughput operations that insert and retrieve documents based on insertion order. 

### Documents

MongoDB 將資料紀錄為 BSON 文件.

`{field: value}`

- field 為 string
- value 為 BSON

#### Dot Notation

- Arrays
- Embedded document

#### BSON type

https://docs.mongodb.com/manual/reference/bson-types/

## Mongo Shell

```sh
# Help
mongo --help
# Run
mongo
mongo --port 28015
mongo "mongodb://mongodb0.example.com:28015"
mongo --host mongodb0.example.com:28015
mongo --host mongodb0.example.com --port 28015

## auth
mongo "mongodb://alice@mongodb0.examples.com:28015/?authSource=admin"
mongo --username alice --password --authenticationDatabase admin --host mongodb0.examples.com --port 28015

## replica set
mongo "mongodb://mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017/?replicaSet=replA"
mongo "mongodb+srv://server.example.com/"
mongo --host replA/mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017

## TLS/SSL
mongo "mongodb://mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017/?replicaSet=replA&ssl=true"
mongo "mongodb+srv://server.example.com/"
mongo --ssl --host replA/mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017
```

```js
// mongo shell 簡單介紹

// db
db
use <database>
db.getSiblingDB(<database>) 
                
// 建立 DB & Collection
use myNewDatabase
db.myCollection.insertOne( { x: 1 } );
db.createCollection(<collection>) // 可以用來建立名字包含空格、底線、與內建函式衝突的集合

// print
db.myCollection.find().pretty()
print()
printjson()

// tab completion
db.myCollection.c<Tab>

//.mongorc.js 預載的設定黨
    
// quit
quit()
<Ctrl-C>
```

mongosh 新的 mongo shell 

- Improved syntax highlighting.
- Improved command history.
- Improved logging.

### Configure the `mongo` Shell

```js
// Display Number of operations
cmdCount = 1;
prompt = function() {
             return (cmdCount++) + "> ";
         }

// Display Database and Hostname
host = db.serverStatus().host;

prompt = function() {
             return db+"@"+host+"$ ";
         }

// Display Up Time and Document Count
prompt = function() {
           return "Uptime:"+db.serverStatus().uptime+" Documents:"+db.stats().objects+" > ";
         }

// Set editor
export EDITOR=vim
mongo

function myFunction () { }
edit myFunction
myFunction

// Change the mongo Shell Batch Size(default = 20)
DBQuery.shellBatchSize = 10;
```

### Shell Help

```js
// shell help
help

// db help
show db
db.help()
db.<function> // show method code
    
// collection Help
show collections
db.collection.help()
db.collection.<function> // show method code
    
// cursor help
db.collection.find().help()
db.collection.find().<function> // show method code
// e.g.
db.collection.find().forEach
```

### Write Scripts for the mongo Shell

```js
// Connections

// (1)
conn = new Mongo();
db = conn.getDB("myDatabase");

// (2)
db = connect("localhost:27020/myDatabase");

// show dbs
db.adminCommand('listDatabases')
// use<db>
db = db.getSiblingDB('<db>')
// show collections
db.getCollectionNames()
// show users
db.getUsers()
// show roles
db.getRoles({showBuiltinRoles: true})
// show logs
db.adminCommand({ 'getLog' : '*' })
// it 
cursor = db.collection.find()
if ( cursor.hasNext() ){
   cursor.next();
}
// print
cursor = db.collection.find();
while ( cursor.hasNext() ) {
   printjson( cursor.next() );
}

```

```shell
## Script with mongo command
mongo test --eval "printjson(db.getCollectionNames())"
mongo localhost:27017/test script/myjsfile.js
```

```js
// Script in mongo shell
load("script/myjsfile.js")
```

### Data Types in the mongo Shell

- Date
  - `Date()` -> current date as String
  - `new Date()`->`Date` object
  - `ISODate()`->`Date` object

```js
var myDateString = Date();
myDateString
typeof myDateString

var myDate = new Date();
myDate instanceof Date

var myDateInitUsingISODateWrapper = ISODate();
myDateInitUsingISODateWrapper instanceof Date
```

- ObjectID

```js
new ObjectId()
```

- Number
  - mongo shell 默認將所有數字視為  double
  - NumberLong(64 bits integer) 
  - NumberInt(32 bits integer)
  - NumberDecimal(128 bits, 10進制)

```js
db.collection.insertOne( { _id: 10, calc: NumberLong("2090845886852") } )

// set
db.collection.updateOne( { _id: 10 },
                      { $set:  { calc: NumberLong("2555555000000") } } )

// increase NumberInt
db.collection.updateOne( { _id: 10 },
                      { $inc: { calc: NumberInt("5") } } )
// find -> NumberLong
db.collection.findOne( { _id: 10 } )

// increase number
db.collection.updateOne( { _id: 10 },
                      { $inc: { calc: 5 } } )
// find -> number
db.collection.findOne( { _id: 10 } )

// decimal 建議用引號包好才不會失去精度
NumberDecimal("1000.55")
NumberDecimal("1000.55000000000")
NumberDecimal(9999999.4999999999)
```

- Double & Decimal: 比較時記得用`NumberDecimal()`轉換double

```js
db.collection.drop()
db.collection.insertMany([
{ "_id" : 1, "val" : NumberDecimal( "9.99" ), "description" : "Decimal" },
{ "_id" : 2, "val" : 9.99, "description" : "Double" },
{ "_id" : 3, "val" : 10, "description" : "Double" },
{ "_id" : 4, "val" : NumberLong("10"), "description" : "Long" },
{ "_id" : 5, "val" : NumberDecimal( "10.0" ), "description" : "Decimal" }])

// 2
db.collection.find({"val": 9.99})
// 1
db.collection.find({"val": NumberDecimal("9.99")})
// 345
db.collection.find({"val": 10})
// 345
db.collection.find({"val": NumberDecimal("10")})
// Checking for decimal Type
db.collection.find({val: {$type: "decimal"}})
```

- instanceof & typeof

```js
a = new ObjectId()
a instanceof ObjectId
typeof a
```

## MongoDB CRUD Operations

所有寫的操作(CUD)都具有原子性。

### CREARE

- db.collection.insertOne()
- db.collection.insertMany()
- db.collection.insert()
- `_id`: 每個 doc 都需要一個 id 當 PK，如果在 insert 忽略時會自動產生 ObjectID 給他

```js
/* Create */

// data
db.inventory.drop()
// Insert a Single Document
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
// Insert Multiple Document
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```

### READ

- db.collection.find()
- db.collection.findone()
- find() return a cursor

```js
// data
db.inventory.drop()
db.inventory.insertMany([
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, tags: ["blank", "red"],  dim_cm: [ 14, 21 ], status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, tags: ["red", "blank"], dim_cm: [ 14, 21 ], status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ], status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ], status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, tags: ["blue"], dim_cm: [ 10, 15.25 ], status: "A" }
]);
// Select All Documents in a Collection
db.inventory.find( {} )

// Specify Equality Condition
db.inventory.find( { status: "D" } )

// Specify Conditions Using Query Operators
db.inventory.find( { status: { $in: [ "A", "D" ] } } )

// Specify AND Conditions
db.inventory.find( { status: "A", qty: { $lt: 30 } } )

// Specify OR Conditions
db.inventory.find( { $or: [ { status: "A" }, { qty: { $gt: 30 } } ] } )

// Specify AND as well as OR Conditions
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```

```js
// Match an Embedded/Nested Document(exact match)
db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } )
db.inventory.find(  { size: { w: 21, h: 14, uom: "cm" } }  )

// Query on Nested Field
db.inventory.find( { "size.uom": "in" } )

// Specify Match using Query Operator
db.inventory.find( { "size.h": { $lt: 15 } } )

// Specify AND Condition
db.inventory.find( { "size.h": { $lt: 15 }, "size.uom": "in", status: "D" } )
```

```js
// Match an array(exact)
db.inventory.find( { tags: ["red", "blank"] } )

// Match an array(contain)
db.inventory.find( { tags: { $all: ["red", "blank"] } } )

// Query an Array for an Element
db.inventory.find( { tags: "red" } )
db.inventory.find( { dim_cm: { $gt: 25 } } )

// Specify Multiple Conditions for Array Elements

// Query an Array with Compound Filter Conditions on the Array Elements
// 單個元素滿足這些條件或任何元素的組合滿足這些條件
db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } )

// Query for an Array Element that Meets Multiple Criteria
// 至少一個元素滿足所有指定的標準
db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )

// Query for an Element by the Array Index Position
db.inventory.find( { "dim_cm.1": { $gt: 25 } } )

// Query an Array by Array Length
db.inventory.find( { "tags": { $size: 3 } } )
```

```js
// data
db.inventory.drop()
db.inventory.insertMany( [
   { item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
   { item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
   { item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
   { item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);

// Query for a Document Nested in an Array(exact match including field order!!)
db.inventory.find( { "instock": { warehouse: "A", qty: 5 } } )

// Specify a Query Condition on a Field Embedded in an Array of Documents
db.inventory.find( { 'instock.qty': { $lte: 20 } } )
db.inventory.find( { 'instock.0.qty': { $lte: 20 } } )

// Specify Multiple Conditions for Array of Documents

// A Single Nested Document Meets Multiple Query Conditions on Nested Fields¶
// 至少一個元素滿足所有指定的標準
db.inventory.find( { "instock": { $elemMatch: { qty: 5, warehouse: "A" } } } )
db.inventory.find( { "instock": { $elemMatch: { qty: { $gt: 10, $lte: 20 } } } } )

// Combination of Elements Satisfies the Criteria
// 單個元素滿足這些條件或任何元素的組合滿足這些條件
db.inventory.find( { "instock.qty": { $gt: 10,  $lte: 20 } } )
db.inventory.find( { "instock.qty": 5, "instock.warehouse": "A" } )
```

```js
// data
db.inventory.drop()
db.inventory.insertMany( [
  { item: "journal", status: "A", size: { h: 14, w: 21, uom: "cm" }, instock: [ { warehouse: "A", qty: 5 } ] },
  { item: "notebook", status: "A",  size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "C", qty: 5 } ] },
  { item: "paper", status: "D", size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "A", qty: 60 } ] },
  { item: "planner", status: "D", size: { h: 22.85, w: 30, uom: "cm" }, instock: [ { warehouse: "A", qty: 40 } ] },
  { item: "postcard", status: "A", size: { h: 10, w: 15.25, uom: "cm" }, instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);

// Return All Fields in Matching Documents
db.inventory.find( { status: "A" } )

// Return the Specified Fields and the _id Field Only
db.inventory.find( { status: "A" }, { item: 1, status: 1 } )

// Suppress _id Field
db.inventory.find( { status: "A" }, { item: 1, status: 1, _id: 0 } )

// Return All But the Excluded Fields
db.inventory.find( { status: "A" }, { status: 0, instock: 0 } )

// Return Specific Fields in Embedded Documents
db.inventory.find(
   { status: "A" },
   { item: 1, status: 1, "size.uom": 1 }
)

// Suppress Specific Fields in Embedded Documents
db.inventory.find(
   { status: "A" },
   { "size.uom": 0 }
)

// Projection on Embedded Documents in an Array
db.inventory.find( { status: "A" }, { item: 1, status: 1, "instock.qty": 1 } )

// Project Specific Array Elements in the Returned Array
// $elemMatch, $slice, and $ are the only way to project specific elements to include in the returned array.
db.inventory.find( { status: "A" }, { item: 1, status: 1, instock: { $slice: -1 } } )
```

```js
// Iterate a Cursor in the mongo Shell
var myCursor = db.inventory.find({ }, {item: 1, status: 1});
while (myCursor.hasNext()) {
   print(tojson(myCursor.next()));
}

var myCursor = db.inventory.find({ }, {item: 1, status: 1});
while (myCursor.hasNext()) {
   printjson(myCursor.next());
}

var myCursor = db.inventory.find({ }, {item: 1, status: 1});
myCursor.forEach(printjson);
```

```js
// data
db.inventory.insertMany([
   { _id: 1, item: null },
   { _id: 2 }
])

// Query for Null or Missing Fields
db.inventory.find( { item: null } )

// Type Check(BSON 10 = Number)
db.inventory.find( { item : { $type: 10 } } )

// Existence Check
db.inventory.find( { item : { $exists: false } } )
```

### UPDATE

- db.collection.updateOne()
- db.collection.updateMany()
- db.collection.replaceOne()
- db.collection.update()
- 不能update `_id`
- 保留欄位順序
- 可以使用 "upsert: true"，沒有match的話就insert

```js
// data
db.inventory.drop()
db.inventory.insertMany( [
   { item: "canvas", qty: 100, size: { h: 28, w: 35.5, uom: "cm" }, status: "A" },
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "mat", qty: 85, size: { h: 27.9, w: 35.5, uom: "cm" }, status: "A" },
   { item: "mousepad", qty: 25, size: { h: 19, w: 22.85, uom: "cm" }, status: "P" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
   { item: "sketchbook", qty: 80, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "sketch pad", qty: 95, size: { h: 22.85, w: 30.5, uom: "cm" }, status: "A" }
] );

// Update a Single Document(If lastModified field does not exist, $currentDate will create the field. See $currentDate for details.)
db.inventory.updateOne(
   { item: "paper" },
   { 
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)

// Update Multiple Documents
db.inventory.updateMany(
   { "qty": { $lt: 50 } },
   {
     $set: { "size.uom": "in", status: "P" },
     $currentDate: { lastModified: true }
   }
)

// Replace a Document
db.inventory.replaceOne(
   { item: "paper" },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
```

- Updates with Aggregation Pipeline(4.2)
  - stage: `$addFo`, `$addFields`, `$set`, `$project`, `$unset`, `$replaceRoot`,  `$replaceWith`
- 可以讓Update語句更靈活，比如根據當前字段的值來表達有條件的更新，或者使用另一個欄位的值來更新一個欄位。

```js
// Example 1(add NOW)
db.students.drop()
db.students.insertMany([
   { _id: 1, test1: 95, test2: 92, test3: 90, modified: new Date("01/05/2020") },
   { _id: 2, test1: 98, test2: 100, test3: 102, modified: new Date("01/05/2020") },
   { _id: 3, test1: 95, test2: 110, modified: new Date("01/04/2020") }
])
db.students.find()
db.students.updateOne( { _id: 3 }, [ { $set: { "test3": 98, modified: "$$NOW"} } ] )
db.students.find()

// Example 2(set default values)
db.students2.drop()
db.students2.insertMany([
   { "_id" : 1, quiz1: 8, test2: 100, quiz2: 9, modified: new Date("01/05/2020") },
   { "_id" : 2, quiz2: 5, test1: 80, test2: 89, modified: new Date("01/05/2020") },
])
db.students2.find()
db.students2.updateMany( {},
  [
    { $replaceRoot: { newRoot:
       { $mergeObjects: [ { quiz1: 0, quiz2: 0, test1: 0, test2: 0 }, "$$ROOT" ] }
    } },
    { $set: { modified: "$$NOW"}  }
  ]
)
db.students2.find()


// Example 3(算平均和等級)
db.students3.drop()
db.students3.insert([
   { "_id" : 1, "tests" : [ 95, 92, 90 ], "modified" : ISODate("2019-01-01T00:00:00Z") },
   { "_id" : 2, "tests" : [ 94, 88, 90 ], "modified" : ISODate("2019-01-01T00:00:00Z") },
   { "_id" : 3, "tests" : [ 70, 75, 82 ], "modified" : ISODate("2019-01-01T00:00:00Z") }
]);

db.students3.updateMany(
   { },
   [
     { $set: { average : { $trunc: [ { $avg: "$tests" }, 0 ] }, modified: "$$NOW" } },
     { $set: { grade: { $switch: {
                           branches: [
                               { case: { $gte: [ "$average", 90 ] }, then: "A" },
                               { case: { $gte: [ "$average", 80 ] }, then: "B" },
                               { case: { $gte: [ "$average", 70 ] }, then: "C" },
                               { case: { $gte: [ "$average", 60 ] }, then: "D" }
                           ],
                           default: "F"
     } } } }
   ]
)
db.students3.find()

// Example 4( 新增陣列元素)
db.students4.drop()
db.students4.insertMany([
  { "_id" : 1, "quizzes" : [ 4, 6, 7 ] },
  { "_id" : 2, "quizzes" : [ 5 ] },
  { "_id" : 3, "quizzes" : [ 10, 10, 10 ] }
])
db.students4.find()
db.students4.updateOne( { _id: 2 },
  [ { $set: { quizzes: { $concatArrays: [ "$quizzes", [ 8, 6 ]  ] } } } ]
)
db.students4.find()

// Example 5(攝氏轉華氏)
db.temperatures.drop()
db.temperatures.insertMany([
  { "_id" : 1, "date" : ISODate("2019-06-23"), "tempsC" : [ 4, 12, 17 ] },
  { "_id" : 2, "date" : ISODate("2019-07-07"), "tempsC" : [ 14, 24, 11 ] },
  { "_id" : 3, "date" : ISODate("2019-10-30"), "tempsC" : [ 18, 6, 8 ] }
])

db.temperatures.find()
db.temperatures.updateMany( { },
  [
    { $addFields: { "tempsF": {
          $map: {
             input: "$tempsC",
             as: "celsius",
             in: { $add: [ { $multiply: ["$$celsius", 9/5 ] }, 32 ] }
          }
    } } }
  ]
)
db.temperatures.find()

```



### DELETE

- db.collection.deleteOne()
- db.collection.deleteMany() 
- delete 不會 drop index
- Atomic

```js
db.inventory.drop()
db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
] );

// Delete All Documents
db.inventory.deleteMany({})

// Delete All Documents that Match a Condition
db.inventory.deleteMany({ status : "A" })

// Delete Only One Document(First) that Matches a Condition
db.inventory.deleteOne( { status: "D" } )
```

### Bulk Write Operations

- 操做可以 Ordered(default), 也可以 Unordered(parallel)
- 支援以下操作
    - insertOne
    - updateOne
    - updateMany
    - replaceOne
    - deleteOne
    - deleteMany
- [Strategies for Bulk Inserts to a Sharded Collection](https://docs.mongodb.com/manual/core/bulk-write-operations/#strategies-for-bulk-inserts-to-a-sharded-collection)

```js
db.characters.drop()
db.characters.insertMany([
{ "_id" : 1, "char" : "Brisbane", "class" : "monk", "lvl" : 3 },
{ "_id" : 2, "char" : "Eldon", "class" : "alchemist", "lvl" : 3 },
{ "_id" : 3, "char" : "Meldane", "class" : "ranger", "lvl" : 3 }
]);

try {
   db.characters.bulkWrite(
      [
         { insertOne :
            {
               "document" :
               {
                  "_id" : 4, "char" : "Dithras", "class" : "barbarian", "lvl" : 4
               }
            }
         },
         { insertOne :
            {
               "document" :
               {
                  "_id" : 5, "char" : "Taeln", "class" : "fighter", "lvl" : 3
               }
            }
         },
         { updateOne :
            {
               "filter" : { "char" : "Eldon" },
               "update" : { $set : { "status" : "Critical Injury" } }
            }
         },
         { deleteOne :
            { "filter" : { "char" : "Brisbane" } }
         },
         { replaceOne :
            {
               "filter" : { "char" : "Meldane" },
               "replacement" : { "char" : "Tanys", "class" : "oracle", "lvl" : 4 }
            }
         }
      ]
   );
}
catch (e) {
   print(e);
}
```

### [Retryable Writes](https://docs.mongodb.com/manual/core/retryable-writes/)

允許MongoDB在遇到網路錯誤，或者在 Replica set 或 sharded cluster 中找不到健康的primary時，自動重試寫入操作一次。

### [Retryable Reads](https://docs.mongodb.com/manual/core/retryable-reads/)

允許MongoDB在遇到某些網路或賜福器錯誤時，自動重試讀取操作一次。

### SQL to MongoDB Mapping Chart

- table -> collection
- row -> document
- column -> field
- join -> lookup
- ...[Ref](https://docs.mongodb.com/manual/reference/sql-comparison/)

| SQL  | MongoDB |
| ---- | ------- |
|      |         |

