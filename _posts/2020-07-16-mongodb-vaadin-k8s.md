---
layout:     post
title:      "MongoDB, Vaadin and K8s Fun"
sub_title:  "Baking with MongoDB"
date:       2020-07-16 12:00:00 -0600
categories: mongodb vaadin kubernetes
---

An experiment in replatforming from SQL to MongoDB using the Vaadin Bakery demo app (and of course, running in a container on K8s).

The topic of when should I use MongoDB comes up **A LOT** and my answer more and more defaults to everything.  This usually elicits some eye rolls but I stand behind it.  Folk's perspective on MongoDB is typically a bit dated and their opinion is far from viewing it as a general purpose database, rather most see it as a niche database or lumped into the NoSQL soup of databases.

MongoDB has come a long way, and for over a decade been delivering value in the core database.  The world runs on Relational Databases no doubt, but will this be the case in 10, 20 and 30 years?  The notion of "applications" is still relatively new, 30 years ago hardly anyone was building "apps" but these days the term "app" is ubiquitous.

So is there a better database for "apps" or in general?  I believe the answer is yes and that MongoDB is the defacto modern database for "apps" and Translytical workloads.  That said, you're way more likely to come across "apps" using a Tabular database instead of MongoDB in the wild.  I recently came across such a thing when perusing [Vaadin](https://vaadin.com/) and noticed relational databases being used in demos...so I thought "hey let's put this to the test" and implement an experiment with MongoDB.  The remainder of this post highlights using MongoDB with [Vaadin's Bakery Demo](https://vaadin.com/docs/bakeryflow/overview.html) application and calls out areas where changes were made.

## The Vaadin Bakery App

Why Vaadin?  Why not...it's solid especially for Java developers.

One of the marquee demo apps is the Vaadin Bakery which documents [changing the database](https://vaadin.com/docs/v14/bakeryflow/changing-database.html) to support H2 (development), MySQL and Postgres.  It's worth noting the demo uses Spring Boot for application scaffolding as well as Spring Data JPA for database access.  One change was replacing Spring Data JPA with Spring Data MongoDB.  I chose to leave most of the Spring Data abstractions in place but also work with the MongoDB Java API in several places.

### Highlevel Changes

* Replace Spring Data JPA with Spring Data MongoDB
* Refactor SQL queries for the Dashboard into MongoDB Aggregations
* Enhance the App by updating the Dashboard model when a Change Stream fires


### Schema

![RDBMS Schema](/assets/images/mongodb-vaadin-k8s/bakery-rdbms-schema.png "RDBMS schema")









































## References

* [Vaadin](https://vaadin.com)
* [Vaadin's live Bakery demo](https://bakery-flow.demo.vaadin.com/)
