---
layout: post
title:  "MongoDB Write Concerns"
sub_title: "Its complicated"
date:   2020-08-08 12:00:00 -0600
categories: mongodb
---

## Majority Write Concern

A client can specify they want a "majority" writeConcern which blocks until a majority of Members have acknowledged the write operation.  Once complete a client use a readConcern of "majority" to read committed data.

[wtimeout](https://docs.mongodb.com/manual/reference/write-concern/#wtimeout) causes a write operation to return a writeConcernError if it does not receive acknowledgment from the required number of Members.  When this occurs MongoDB does not undo successful data modifications performed before the writeConcern exceeded the wtimeout limit.  When a write operation times out the [data may exist](https://docs.mongodb.com/manual/core/replica-set-write-concern/#verify-write-operations-to-replica-sets) on a subset of Members at the time the writeConcernError is raised.  Replication continues until all Members have the data.  Applications should take into account the potential availability of written data regardless of the state of writeConcern acknowledgment.

If a writeConcern majority operation receives a writeConcernError due to a timeout and the data eventually replicates (perhaps milliseconds after raising the writeConcernError) then what should the application do?

The more Members that acknowledge the write, the less likely writes could rollback if the primary fails.

What is the purpose of writeConcern majority, what value does it provide or what does it protect against?

Consider the following scenario when using a writeConcern=1

1. Primary writes 10 Documents to the oplog
2. Primary dies
3. Secondary-1 applied 5 of 10 changes before Primary died
4. Secondary-2 applied 7 of 10 changes before Primary died
5. Secondary-2 wins the election because its more current
6. Previous Primary comes back online as a Secondary
7. The 3 writes not yet replicated are rolled back

Risk: Application thinks these Documents were written yet only 7 of 10 actually were.

```javascript
var s1 = db.getMongo().startSession();
var products = s1.getDatabase('test').getCollection('products');
s1.startTransaction({readConcern: {level: 'snapshot'}, writeConcern: {w: 'majority', wtimeout: 1}});
products.insert({ item: "candy", qty : 10, type: "kisses" });
s1.commitTransaction();





















## References
