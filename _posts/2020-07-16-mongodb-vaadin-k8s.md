---
layout:     post
title:      "MongoDB, Vaadin and K8s Fun"
sub_title:  "Baking with MongoDB"
date:       2020-07-16 12:00:00 -0600
categories: mongodb vaadin kubernetes
---

An experiment in replatforming from SQL to MongoDB using the Vaadin Bakery demo app (and of course, running in a container on K8s).

The topic of when should I use MongoDB comes up **A LOT** and my answer more and more defaults to everything.  This usually elicits some eye rolls but I stand behind it.  Folk's perspective on MongoDB is typically a bit dated and their opinion is far from considering it a general purpose database, rather most see it as a niche database or lumped into the NoSQL soup of specialty databases.

MongoDB has come a long way, and for over a decade been delivering value in the core database.  The world runs on Relational Databases no doubt, but will this be the case in 10, 20 and 30 years?  Consider the notion of "apps" is still relatively new, 30 years ago hardly anyone was building "apps" but these days the term "app" is ubiquitous.

So is there a better database for "apps" or in general?  I believe the answer is yes and that MongoDB is the defacto modern database for "apps" and Translytical workloads.  That said, you're way more likely to come across "apps" using a Tabular database instead of MongoDB in the wild.  I recently came across such a thing when perusing [Vaadin](https://vaadin.com/) and noticed relational databases being used in demos...so I thought "hey let's put this to the test" and implement an experiment with MongoDB.  The remainder of this post highlights using MongoDB with [Vaadin's Bakery Demo](https://vaadin.com/docs/bakeryflow/overview.html) application and calls out areas where changes were made.

## The Vaadin Bakery App

Why Vaadin?  Why not...it's solid especially for Java developers.

One of the marquee demo apps is the Vaadin Bakery which documents [changing the database](https://vaadin.com/docs/v14/bakeryflow/changing-database.html) to support H2 (development), MySQL and Postgres.  It's worth noting the demo uses Spring Boot for application scaffolding as well as Spring Data JPA for database access.  One change was replacing Spring Data JPA with Spring Data MongoDB.  I chose to leave most of the Spring Data abstractions in place but also work with the MongoDB Java API in several places.

## High-level Changes

* Replace Spring Data JPA with [Spring Data MongoDB](https://spring.io/projects/spring-data-mongodb)
* Refactor SQL queries for the Dashboard into [MongoDB Aggregations](https://docs.mongodb.com/manual/aggregation/)
* Enhance the App by updating the Dashboard model when a [Change Stream](https://docs.mongodb.com/manual/changeStreams/) fires

## Dashboard Aggregations

MongoDB is capable of performing rich queries including aggregations which is required to perform groupings for the Dashboard.  This is the area where I had the most refactoring of the Bakery App.  Essentially removing the count and sum queries from `OrderRepository` and creating a new `DashboardService` class containing the aggregations to support the `DashboardView`.

### Deliveries Per Year

This aggregation supports the "Deliveries per Year" bar chart by matching "Delivered" orders for the given year and summing the orders per month.  Functionally equivalent to `SELECT FROM WHERE GROUP-BY` in SQL.

```javascript
db.orders.aggregate(
  [{$match: {
      state: "DELIVERED",
      dueDate: {
        $gte: ISODate("2020-01-01"), $lte: ISODate("2020-12-31")
      }}
   },
   {$group: { _id: { $month: "$dueDate" }, count: {$sum: 1}}},
   {$sort:  { _id: 1 }}
 ])

// sample output
{ "_id" : 1, "count" : 150 }
{ "_id" : 2, "count" : 172 }
{ "_id" : 3, "count" : 143 }
{ "_id" : 4, "count" : 165 }
{ "_id" : 5, "count" : 138 }
{ "_id" : 6, "count" : 155 }
{ "_id" : 7, "count" : 167 }
{ "_id" : 8, "count" : 21 }
```

### Deliveries Per Month

Similar to "Deliveries Per Year" except the grouping occurs across the days in any given month.

```javascript
db.orders.aggregate(
  [{$match: {
      state: "DELIVERED",
      dueDate: {
        $gte: ISODate("2020-07-01"), $lte: ISODate("2020-07-31")
      }}
   },
   {$group: { _id: { $dayOfMonth: "$dueDate" }, count: { $sum: 1 }}},
   {$sort:  { _id: 1 }}
 ])

// sample output
{ "_id" : 1, "count" : 7 }
{ "_id" : 2, "count" : 1 }
{ "_id" : 3, "count" : 10 }
// ...
{ "_id" : 28, "count" : 3 }
{ "_id" : 29, "count" : 7 }
{ "_id" : 30, "count" : 5 }
```

### Yearly Sales Totals

This aggregation computes the Sales Totals for the last 3 years.  We use the `unwind` array operator to flatten the items array then group on month and year, summing the dollar amount sold.  The amount sold is computed by multiplying the quantity purchased times the product price.

```javascript
db.orders.aggregate(
  [{$match: {
      state: "DELIVERED",
      dueDate: {
        $gte: ISODate("2018-01-01"), $lte: ISODate("2020-12-31")
      }}
  },
  {$project: {
     year:  { $year: "$dueDate"  },
     month: { $month: "$dueDate" },
     items: 1
  }},
  {$unwind:  { path: "$items"}},
  {$project: { year: 1, month: 1, items: 1,
     total: {
       $multiply:[ "$items.quantity", "$items.product.price" ]
     }
  }},
  {$group: {
     _id :  { month : "$month", year : "$year" },
     month: { $first: "$month" },
     year:  { $first: "$year"  },
     sales: { $sum:   "$total" },
  }},
  {$sort: {year: -1, month: -1}}
])

// sample output
{ "_id" : { "month" : 8, "year" : 2020 }, "month" : 8, "year" : 2020, "sales" : 1058638 }
{ "_id" : { "month" : 7, "year" : 2020 }, "month" : 7, "year" : 2020, "sales" : 9038046 }
{ "_id" : { "month" : 6, "year" : 2020 }, "month" : 6, "year" : 2020, "sales" : 8145360 }
{ "_id" : { "month" : 5, "year" : 2020 }, "month" : 5, "year" : 2020, "sales" : 8855579 }
{ "_id" : { "month" : 4, "year" : 2020 }, "month" : 4, "year" : 2020, "sales" : 8778870 }
{ "_id" : { "month" : 3, "year" : 2020 }, "month" : 3, "year" : 2020, "sales" : 8340463 }
{ "_id" : { "month" : 2, "year" : 2020 }, "month" : 2, "year" : 2020, "sales" : 9241052 }
{ "_id" : { "month" : 1, "year" : 2020 }, "month" : 1, "year" : 2020, "sales" : 7959236 }
// ...
{ "_id" : { "month" : 3, "year" : 2018 }, "month" : 3, "year" : 2018, "sales" : 8981794 }
{ "_id" : { "month" : 2, "year" : 2018 }, "month" : 2, "year" : 2018, "sales" : 8416290 }
{ "_id" : { "month" : 1, "year" : 2018 }, "month" : 1, "year" : 2018, "sales" : 8824770 }
```

### Products Delivered Per Month

This aggregation computes the quantity of products delivered per month by grouping on product name.

```javascript
db.orders.aggregate(
  [{$match: {
      state: "DELIVERED",
      dueDate: {
        $gte: ISODate("2020-07-01"),
        $lte: ISODate("2020-07-31")
   }}},
   {$unwind: { path: "$items" }},
   {$unwind: { path: "$items.product"}},
   {$group: {
      _id: "$items.product.name",
      quantity: { $sum: { $multiply: [ "$items.quantity", 1 ]}},
   }},
   {$sort: {quantity: 1}}
 ])
// sample output
{ "_id" : "Strawberry Tart", "quantity" : 7 }
{ "_id" : "Strawberry Bun", "quantity" : 49 }
{ "_id" : "Strawberry Cheese Cake", "quantity" : 53 }
{ "_id" : "Vanilla Cracker", "quantity" : 173 }
{ "_id" : "Raspberry Blueberry Tart", "quantity" : 226 }
{ "_id" : "Vanilla Pastry", "quantity" : 367 }
{ "_id" : "Blueberry Cheese Cake", "quantity" : 398 }
{ "_id" : "Vanilla Blueberry Cake", "quantity" : 478 }
```

### Screenshot of the DashboardView

The aggregations above support the `DashboardView` that previously relied on SQL queries.

![DashboardView](/assets/images/mongodb-vaadin-k8s/dashboard-view.png "DashboardView")

## Push Data to the UI with Change Streams

Change Streams is a feature of MongoDB that puts data in motion and allows your applications to react when data changes.  The Bakery App has been enhanced with a Change Stream that pushes data to the Vaadin application when an Order is inserted, updated or deleted.  In response to the Change Stream the app re-renders the Dashboard with the latest information.

```java
// ChangeStreamCommand.java - code omitted for readability
private void openChangeStream() {
    MongoCollection<Document> collection = database.getCollection("orders");
    collection.watch(asList(
        Aggregates.match(Filters.in("operationType",
            asList("insert", "update", "replace", "delete")))))
                .forEach((Consumer<ChangeStreamDocument<Document>>) document -> {
                    Broadcaster.broadcast(document.toString());
            });
}

// MainView.java - code omitted for readability
@Push
public class MainView extends AppLayout {

  @Override
  protected void onAttach(AttachEvent attachEvent) {
      currentUi = attachEvent.getUI();
      Broadcaster.register(document -> {
          currentUi.access(() -> {
              if (this.getContent() instanceof DashboardView) {
                  DashboardView dashboardView = (DashboardView) this.getContent();
                  dashboardView.refreshData();
              }
          });
      });
  }
}
```

## Containerize and K8s Deployment






































## References

* [Vaadin](https://vaadin.com)
* [Vaadin's live Bakery demo](https://bakery-flow.demo.vaadin.com/)
