---
layout: post
title:  "MongoDB Cert Guide"
sub_title: "A mini guide to mongo certification"
date:   2020-07-26 12:00:00 -0600
categories: mongodb certification
---

MongoDB is a nuanced technology and thus clearing the exam will take serious prep time.

I know several folks that I would consider MongoDB experts that failed a couple times before clearing.  Prepare well, read the docs and do the practice exams over and over until you're comfortable has been the feedback.   That said this kit is a curated list of things you need to know in order to clear.

## CRUD

* Understand create, read, update and delete operations in MQL
* Know MongoDB data types
* Use insert, save, update and findAndModify to create documents
* Match insert commands with descriptions of what they do
* Know how to perform bulk inserts
* Understand uniqueness constraint of `_id` fields for CRUD operations
* Understand how ObjectIds are created and used

### Create

#### db.collection.insert()

```javascript
db.collection.insert(
   <document or array of documents>, {
     writeConcern: <document>,
     ordered: <boolean>
   }
)
```

* writeConcern - optional, omit to use default writeConcern.  DO NOT explicitly set if operation runs in a Transaction.  writeConcern=1 is default for MongoDB.
* ordered - optional, true will fail fast, false will try all, default is true.
* If the collection does not exist `insert()` will create one
* If `_id` is not provided MongoDB will add one before insert.  Most drivers create an ObjectId, however mongod will create if one is not provided.
* Transactions - `insert()` can be used inside a multi-document transaction.  The collection must already exist, an insert operation that would result in creating a new collection are NOT allowed in a Transaction.

```javascript
// _id ObjectId created
db.products.insert({item: "card", qty: 15});
// _id specified in the insert
db.products.insert({_id: 1, item: "box", qty: 20})
```

* Inserting multiple documents - default is ordered

```javascript
// mongodb bulk inserts in order by default
db.products.insert([
  {_id: 11, item: "pencil", qty: 50, type: "no. 2"},
  {item: "pen", qty: 20},
  {item: "eraser", qty: 25}])
```

* Unordered Insert - continues processing if error occurs

```javascript
// _id: 11 already exists but insert continues processing
db.products.insert([
     { _id: 11, item: "lamp", qty: 50, type: "desk" },
     { _id: 21, item: "lamp", qty: 20, type: "floor" },
     { _id: 22, item: "bulk", qty: 100 }],
   { ordered: false }
)
```

* Override default WriteConcern

```javascript
// insert and wait for ack from majority of voting members
db.products.insert(
    { item: "envelopes", qty : 100, type: "Clasp" },
    { writeConcern: { w: "majority", wtimeout: 5000 } }
)
```

* db.collection.save()
* db.collection.update()
* db.collection.findAndModify()

## Indexes

## Data Modeling

## Aggregation

## Replication

## Sharding

## Tooling
