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
