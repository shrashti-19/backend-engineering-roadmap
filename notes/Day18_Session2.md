# Day 18 — Session 2: Creating & Using MongoDB Indexes

## Overview

In Session 1, I learned why indexes are needed and how they can help MongoDB avoid scanning a large collection.

In Session 2, I started learning **how to create indexes in Mongoose**, how to choose fields for indexing, and what `1` and `-1` mean in an index definition.

---

# 1. Creating an Index in Mongoose

An index can be created on a Mongoose schema using `.index()`.

```javascript
promptSchema.index({ category: 1 });
```

This creates an index on the `category` field.

### Breakdown

```text
promptSchema
     ↓
The schema on which we create the index

.index()
     ↓
Mongoose method used to define an index

{ category: 1 }
     ↓
Field to index + index ordering
```

---

# 2. Single-Field Index

An index can be created on a single field.

Example:

```javascript
promptSchema.index({ category: 1 });
```

This creates an index on `category`.

Another example:

```javascript
promptSchema.index({ likes: 1 });
```

This creates an index on `likes`.

---

# 3. Meaning of 1 and -1

In an index definition:

```text
1  → ascending order
-1 → descending order
```

Example:

```javascript
promptSchema.index({ likes: 1 });
```

Conceptually, the index is ordered like:

```text
10 → 50 → 100 → 500 → 900
```

Whereas:

```javascript
promptSchema.index({ likes: -1 });
```

is ordered like:

```text
900 → 500 → 100 → 50 → 10
```

### Important

`1` and `-1` describe the **ordering of the index**.

They should not simply be interpreted as:

```text
1  → MongoDB scans the collection upward
-1 → MongoDB scans the collection downward
```

Instead:

> `1` = ascending index ordering
> `-1` = descending index ordering

The index ordering can be useful when MongoDB needs to process data in a particular order, especially for sorting.

---

# 4. Index as a Shortcut

An index can be thought of as a **shortcut/organized lookup structure**.

Without a suitable index:

```text
Query
  ↓
COLLSCAN
  ↓
Potentially examine many documents
  ↓
Find matching documents
```

With a suitable index:

```text
Query
  ↓
Index
  ↓
Locate relevant index entries
  ↓
Find corresponding documents
```

The index does not represent a second copy of the entire collection.

It provides organized information that helps MongoDB locate relevant documents.

---

# 5. Choosing Fields to Index

A field is a good candidate for an index when the application **frequently uses that field in queries**.

For example:

```javascript
Prompt.find({
  category: "Java"
});
```

If the application frequently filters prompts by `category`, an index on `category` may be useful:

```javascript
promptSchema.index({ category: 1 });
```

### Important principle

Don't think:

> "Every field should have an index."

Instead think:

> **"Which fields are frequently involved in queries that need efficient access?"**

Indexes should be based on the application's **query patterns/access patterns**.

---

# 6. Why Not Index Every Field?

Suppose the schema contains:

```javascript
title: String,
category: String,
likes: Number,
description: String
```

If the application never filters, sorts, or searches by `description`, creating:

```javascript
promptSchema.index({ description: 1 });
```

may provide little value.

However, the index still has costs such as:

* Additional storage
* Maintenance overhead
* Extra work during writes

Therefore:

> Indexes should be created strategically rather than blindly indexing every field.

---

# 7. Index + Query Example

Suppose we have:

```javascript
promptSchema.index({ category: 1 });
```

and frequently execute:

```javascript
Prompt.find({
  category: "Java"
});
```

The index can help MongoDB locate the relevant entries more efficiently than potentially scanning the entire collection.

---

# 8. Index Ordering and Sorting

The ordering of an index can also be relevant when sorting.

For example:

```javascript
promptSchema.index({
  likes: -1
});
```

creates an index ordered by `likes` in descending order.

This can be useful when working with queries that need results ordered by likes.

However, the important idea is:

> The index ordering does not mean MongoDB is forced to scan the collection in that direction.

MongoDB can traverse an index as appropriate for the query.

---

# Key Takeaways

```text
.index()          → creates/defines an index

1                 → ascending index order

-1                → descending index order

Good index field  → frequently used in important queries

Index             → organized lookup structure / shortcut

Indexes are NOT   → free
```

### Core Mental Model

```text
Application query
       ↓
Does an index help this query?
       ↓
If yes → MongoDB may use the index
       ↓
Less unnecessary data examination
       ↓
Potentially better query performance
```

### Most Important Principle

> **Don't create indexes just because a field exists. Create them based on the application's query patterns and performance needs.**

# Day 18 — Session 2: Creating & Using MongoDB Indexes

## Overview

In Session 2, I learned how to create indexes in Mongoose, how MongoDB builds and maintains indexes, what an index conceptually stores, how `1` and `-1` define index ordering, and how indexes help queries access data more efficiently.

## 1. Creating an Index in Mongoose

An index can be created on a Mongoose schema using `.index()`.

```
promptSchema.index({ category: 1 });
```

This creates an index on the `category` field.

### Breakdown

```
promptSchema

     ↓

The schema on which we create the index

.index()

     ↓

Mongoose method used to define an index

{ category: 1 }

     ↓

Field to index + index ordering
```

## 2. Single-Field Index

An index can be created on a single field.

Example:

```
promptSchema.index({ category: 1 });
```

This creates an index on `category`.

Another example:

```
promptSchema.index({ likes: 1 });
```

This creates an index on `likes`.

MongoDB automatically creates an index on `_id`, but indexes on other fields generally need to be explicitly created.

## 3. When Is the Index Created?

When an index is created, MongoDB builds the index using the existing data.

After that, MongoDB maintains the index as data changes.

```
Create Index
     ↓
Build index from existing data
     ↓
New document inserted/updated/deleted
     ↓
MongoDB updates the relevant index entries
```

We do not recreate the index every time a document is written.

## 4. What Does an Index Store?

Conceptually, an index stores:

```
Indexed field value → reference to document
```

Example:

```
Documents:

Doc 1 → likes: 500
Doc 2 → likes: 100
Doc 3 → likes: 900
```

For:

```
promptSchema.index({ likes: 1 });
```

the index can be thought of as:

```
100 → Doc 2
500 → Doc 1
900 → Doc 3
```

The index is not a complete duplicate of the documents.

It provides organized information that helps MongoDB locate the corresponding documents efficiently.

## 5. Why Are Indexes Efficient?

Without a suitable index, MongoDB may need to examine many documents:

```
Query
  ↓
COLLSCAN
  ↓
Check many documents
  ↓
Find matching documents
```

With a suitable index:

```
Query
  ↓
Index
  ↓
Locate relevant index entries
  ↓
Fetch corresponding documents
```

The important idea is:

The index structure is organized ahead of time and maintained as data changes, so MongoDB does not have to figure out the lookup structure from scratch for every query.

## 6. Meaning of 1 and -1

In an index definition:

```
1  → ascending order

-1 → descending order
```

Example:

```
promptSchema.index({ likes: 1 });
```

Conceptually, the index is ordered like:

```
100 → 500 → 900
```

Whereas:

```
promptSchema.index({ likes: -1 });
```

is ordered like:

```
900 → 500 → 100
```

### Important

`1` and `-1` describe the ordering of the index.

They should not simply be interpreted as:

```
1  → MongoDB scans the collection upward

-1 → MongoDB scans the collection downward
```

Instead:

`1` = ascending index ordering

`-1` = descending index ordering

## 7. Single-Field Index Can Be Traversed in Either Direction

Suppose we have:

```
promptSchema.index({ likes: 1 });
```

Conceptually:

```
100 → 500 → 900
```

MongoDB can generally traverse this single-field index in either direction:

```
Ascending:

100 → 500 → 900
```

or:

```
Descending:

900 → 500 → 100
```

Therefore, an ascending single-field index can still be useful for a query that needs descending order.

For example:

```
Prompt.find()
  .sort({ likes: -1 })
  .limit(10);
```

The important point is:

A single-field index does not mean MongoDB can only use it in the direction in which it was defined.

## 8. Index as a Shortcut

An index can be thought of as an organized lookup structure or shortcut to the data.

For example:

```
promptSchema.index({ category: 1 });
```

and:

```
Prompt.find({
  category: "Java"
});
```

The category index helps MongoDB locate the relevant Java entries more efficiently.

Think:

```
Index
  ↓
category values
  ↓
Relevant document references
```

The index does not store a second complete copy of the collection.

## 9. Index Is Created on a Field, Not a Specific Value

If we create:

```
promptSchema.index({ category: 1 });
```

the index is created on the `category` field, not specifically on `"Java"`.

For example:

```
category
   ↓
Java
Python
Node.js
C++
...
```

Then a query such as:

```
Prompt.find({
  category: "Java"
});
```

can use the category index to locate the relevant entries efficiently.

## 10. Choosing Fields to Index

A field is a good candidate for an index when the application frequently uses that field in important queries.

For example:

```
Prompt.find({
  category: "Java"
});
```

If the application frequently filters prompts by category, an index on `category` may be useful:

```
promptSchema.index({ category: 1 });
```

### Important principle

Don't think:

> "Every field should have an index."

Instead think:

> "Which fields are frequently involved in queries that need efficient access?"

Indexes should be based on the application's query patterns/access patterns.

## 11. Why Not Index Every Field?

Suppose the schema contains:

```
title: String,
category: String,
likes: Number,
description: String
```

If the application rarely or never queries using `description`, creating:

```
promptSchema.index({ description: 1 });
```

may provide little value.

Indexes have costs such as:

- Additional storage
- Maintenance overhead
- Extra work during writes

Therefore:

Indexes should be created strategically rather than blindly indexing every field.

## 12. Index + Query Example

Suppose we have:

```
promptSchema.index({ category: 1 });
```

and frequently execute:

```
Prompt.find({
  category: "Java"
});
```

The category index can help MongoDB locate the relevant entries more efficiently than potentially scanning the entire collection.

The index primarily helps with the field it was created for.

## 13. Index Does Not Automatically Optimize Every Part of a Query

Consider:

```
promptSchema.index({ category: 1 });

Prompt.find({ category: "Java" })
  .sort({ likes: -1 })
  .limit(10);
```

The query contains:

```
Filter → category
Sort   → likes
Limit  → 10
```

The existing index:

```
{ category: 1 }
```

primarily helps with the category filtering.

It does not automatically mean that sorting by likes is optimized.

Indexes should therefore be designed around the actual query/access pattern.

## 14. Index Ordering and Sorting

The ordering of an index can be useful when queries require data in a particular order.

For example:

```
promptSchema.index({
  likes: -1
});
```

creates an index ordered by likes in descending order.

A query such as:

```
Prompt.find()
  .sort({ likes: -1 })
  .limit(10);
```

can potentially benefit from an index on likes.

For a single-field index, MongoDB can generally traverse the index in either direction.

## 15. Index Trade-offs

Indexes improve read performance but are not free.

```
Faster reads 🚀
     ↕
Extra storage
+
Write / maintenance overhead
```

When documents are inserted, updated, or deleted, MongoDB may also need to update the relevant index entries.

Therefore:

Indexes improve read performance at the cost of additional storage and maintenance work.

## Key Takeaways

```
.index()              → creates/defines an index

_id                   → automatically indexed by MongoDB

1                     → ascending index order

-1                    → descending index order

Index                 → organized lookup structure / shortcut

Index stores          → indexed values + document references

Good index field      → frequently used in important queries

Index creation        → built from existing data

Data changes          → MongoDB maintains relevant indexes

Indexes are NOT free  → storage + write/maintenance cost
```

## Core Mental Model

```
Application Query
       ↓
Does a suitable index exist?
       ↓
MongoDB's query planner may choose it
       ↓
Use organized index entries
       ↓
Locate relevant documents efficiently
```

## Most Important Principle

Don't create indexes just because a field exists. Create them based on the application's query patterns and performance needs.
