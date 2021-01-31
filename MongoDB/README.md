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





