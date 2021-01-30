---
layout:     post
title:      "Atlas Text Search"
sub_title:  "Finding needles in haystacks"
date:       2020-08-09 12:00:00 -0600
categories: mongodb atlas search
---

## Atlas Text Search

MongoDB clusters running in Atlas can take advantage of fine-grained text indexing which enables advanced search capabilities across your documents.

### Sample Index Definition

```json
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "title": {
        "type": "string",
        "analyzer": "lucene.standard",
        "multi": {
          "keywordAnalyzer": {
            "type": "string",
            "analyzer": "lucene.keyword"
          }
        }
      },
      "genres": {
        "type": "string",
        "analyzer": "lucene.standard"
      },
      "plot": {
        "type": "string",
        "analyzer": "lucene.standard"
      }
    }
  }
}
```

### Sample Text Search Queries

Search for the word "baseball" in the plot field.

```javascript
db.movies.aggregate([{
    $search: {
      "text": {
        "query": "baseball",
        "path": "plot"
      }
    }}, { $limit: 5 }, {
    $project: {
      "_id": 0,
      "title": 1,
      "plot": 1
    }}])
```

Use the keywordAnalyzer to search for exact matches on the string "The Count of Monte Cristo" on the title field.

```javascript
db.movies.aggregate([
  {
    $search: {
      "text": {
        "query": "The Count of Monte Cristo",
        "path": { "value": "title", "multi": "keywordAnalyzer" }
      }
    }
  },
  {
    $project: {
      "title": 1,
      "year": 1,
      "_id": 0
    }
  }
])
```

Not using the keywordAnalyzer, implies use of the standard analyzer and finds movies with the word "Count" or "Monte" or "Cristo" in the title.

```javascript
db.movies.aggregate([
  {
    $search: {
      "text": {
        "query": "The Count of Monte Cristo",
        "path": "title"
      }
    }
  },
  {
    $project: {
      "title": 1,
      "year": 1,
      "_id": 0
    }
  }
])
```

Use the compound operator to combine several operators into a single query.

* Plot field should contain Hawaii or Alaska
* Plot field must contain a four digit number
* Genres field must NOT be Comedy or Romance
* Title field must NOT be Beach or Snow

```javascript
db.movies.aggregate([
  {
    $search: {
      "compound": {
        "must": [ {
          "text": {
             "query": ["Hawaii", "Alaska"],
             "path": "plot"
          },
        },
        {
          "regex": {
             "query": "([0-9]{4})",
             "path": "plot",
             "allowAnalyzedField": true
          }
        } ],
        "mustNot": [ {
          "text": {
            "query": ["Comedy", "Romance"],
            "path": "genres"
          }
        },
        {
          "term": {
            "query": ["Beach", "Snow"],
            "path": "title"
          }
        } ]
      }
    }
  },
  {
    $project: {
      "title": 1,
      "plot": 1,
      "genres": 1,
      "_id": 0
    }
  }
])
```

## Baking with MongoDB Text Search Index

Order fields to index

+ orders.customer.fullName
+ orders.customer.details
+ orders.customer.phoneNumber
orders.dueDate
orders.items[].comment
orders.items[].product.name
orders.items[].product.price
orders.pickupLocation
orders.state

```json
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "customer": {
        "type": "document",
        "fields": {
          "fullName": {
            "type": "string",
            "analyzer": "lucene.standard"
          },
          "details": {
            "type": "string",
            "analyzer": "lucene.standard"
          },
          "phoneNumber": {
            "type": "string",
            "analyzer": "lucene.standard"            
          }          
        }
      },
      "items": {
        "type": "string",
        "analyzer": "lucene.standard"
      }
    }
  }
}
```

## Baking with MongoDB Queries

```javascript
db.orders.aggregate([
  {
    $search: {
      "text": {
        "query": ["Lionel", "Gluten"],
        "path": ["customer.fullName", "customer.details"]
      }
    }
  }, { $limit: 5 },
  {
    $project: {
      "customer.fullName": 1,
      "customer.details": 1,
      "state": 1,
      "items.product.name": 1,
      "_id": 0
    }
  }
])

db.orders.aggregate([
  {
    $search: {
      "text": {
        "query": ["cheese", "Nacho"],
        "path": ["customer.fullName", "customer.details"]
      }
    }
  }, { $limit: 5 },
  {
    $project: {
      "customer.fullName": 1,
      "customer.details": 1,
      "customer.phoneNumber": 1,
      "state": 1,
      "items.product.name": 1,
      "_id": 0
    }
  }
])

db.orders.aggregate([
  {
    $search: {
      "text": {
        "query": ["5", "Corbett"],
        "path": ["customer.fullName", "customer.details", "customer.phoneNumber"],
        "fuzzy": {
          "maxEdits": 1,
          "maxExpansions": 100,
        }        
      }
    }
  }, { $limit: 5 },
  {
    $project: {
      "customer.fullName": 1,
      "customer.details": 1,
      "customer.phoneNumber": 1,
      "state": 1,
      "items.product.name": 1,
      "_id": 0
    }
  }
])

db.orders.aggregate([
  {
    $search: {
      "text": {
        "query": "Vanilla",
        "path": "items.product.name"
      }
    }
  }, { $limit: 5 },
  {
    $project: {
      "customer.fullName": 1,
      "customer.details": 1,
      "customer.phoneNumber": 1,
      "state": 1,
      "items.product.name": 1,
      "_id": 0
    }
  }
])


db.fruit.aggregate([
  {
    $search: {
      "compound": {
        "must": [{
          "text": {
            "query": "varieties",
            "path": "description"
          }
        }],
        "mustNot": [{
          "text": {
            "query": "apples",
            "path": "description"
          }
        }]
      }
    }
  }
])

db.orders.aggregate([
  {
    $search: {
      "compound": {
        "must": [{
          "text": {
            "query": "varieties",
            "path": "description"
          }
        }],
        "mustNot": [{
          "text": {
            "query": "apples",
            "path": "description"
          }
        }]
      }
    }
  }
])
```


















## Definitions

1. Analyzers - Controls how the search index is created
1. Index Definition Format - Defines the text index, dynamic and static field mappings
1.





## References

1. [Atlas Search](https://docs.atlas.mongodb.com/atlas-search/)
1. [Not using Atlas?  Try MongoDB text indexing](https://docs.mongodb.com/manual/core/index-text/)
