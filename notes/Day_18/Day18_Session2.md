# Day 18 — Session 2: MongoDB Indexes

## Main Points

- Create a Mongoose index with `.index()`:

  ```javascript
  promptSchema.index({ category: 1 });
  ```

- MongoDB automatically indexes `_id`. Add indexes to other fields only when they support frequent or important queries.

- An index is an organized lookup structure: it maps indexed field values to document references. It is not a full copy of the collection.

- When an index is created, MongoDB builds it from existing documents and maintains it when documents are inserted, updated, or deleted.

- Without a suitable index, MongoDB may perform a collection scan (`COLLSCAN`). With one, it can locate matching documents more efficiently.

- `1` means ascending index order; `-1` means descending index order.

  ```javascript
  promptSchema.index({ likes: -1 });
  ```

- A single-field index can generally be traversed in either direction, so an ascending index may also help a descending sort on that field.

- Indexes should match query patterns. For example, an index on `category` helps a category filter:

  ```javascript
  Prompt.find({ category: "Java" });
  ```

- An index on one field does not automatically optimize another part of a query. For example, `{ category: 1 }` helps filtering by category but does not necessarily optimize sorting by `likes`.

  ```javascript
  Prompt.find({ category: "Java" })
    .sort({ likes: -1 })
    .limit(10);
  ```

- Indexes improve reads, but cost extra storage and add work to writes. Do not index every field—index fields used often in meaningful filters, sorts, and lookups.

## Mental Model

```text
Query → suitable index? → query planner may use it → find matching documents efficiently
```
## Learning Progress

### Day 18 — Session 2 (PART-2): Creating & Using MongoDB Indexes ✅

This session covered creating indexes in Mongoose, how MongoDB builds and maintains them, what they store, index ordering, and how MongoDB decides whether to use them.

#### Creating an index in Mongoose

```javascript
promptSchema.index({ category: 1 });
```

This creates an index on `category`. MongoDB automatically creates an index on `_id`; indexes on other fields generally need to be explicitly created.

```text
promptSchema  → schema on which we create the index
.index()      → Mongoose method used to define an index
{ category: 1 } → field to index + index ordering
```

When an index is explicitly defined, MongoDB builds it from existing data and maintains it after inserts, updates, and deletes:

```text
Define/Create Index → MongoDB builds the index → Data changes → MongoDB maintains the index
```

It is not recreated from scratch for every write. A query also does not create an index—it may use an existing one if the query planner finds it beneficial.

#### What an index stores

Conceptually, an index stores an indexed field value and a reference to its document:

```text
Indexed field value → reference to document
```

For these documents:

```text
Doc 1 → likes: 500
Doc 2 → likes: 100
Doc 3 → likes: 900
```

```javascript
promptSchema.index({ likes: 1 });
```

can be thought of as:

```text
100 → Doc 2
500 → Doc 1
900 → Doc 3
```

An index is not a complete duplicate of the documents. It is a separate organized lookup structure; it does not reorder the collection itself. The collection can remain `Doc 1 → 500`, `Doc 2 → 100`, `Doc 3 → 900` while its index is organized as `100 → Doc 2`, `500 → Doc 1`, `900 → Doc 3`.

Without a suitable index:

```text
Query → COLLSCAN → check many documents → find matching documents
```

With a suitable index:

```text
Query → Index → locate relevant index entries → fetch corresponding documents
```

The lookup structure is organized ahead of time and maintained as data changes, so MongoDB does not need to derive it from scratch for every query.

#### `1` and `-1`: index ordering

`1` means ascending index order; `-1` means descending index order.

```javascript
promptSchema.index({ likes: 1 });  // 100 → 500 → 900
promptSchema.index({ likes: -1 }); // 900 → 500 → 100
```

They do **not** mean MongoDB scans the collection upward or downward. A single-field index can generally be traversed in either direction, so an ascending index can still support a descending sort on the same field:

```javascript
Prompt.find()
  .sort({ likes: -1 })
  .limit(10);
```

#### Choosing index fields

```javascript
promptSchema.index({ category: 1 });
```

This is an index on the `category` field, not specifically on the value `"Java"`; it can contain values such as `Java`, `Python`, `Node.js`, and `C++`. It can help a frequent query such as:

```javascript
Prompt.find({ category: "Java" });
```

A good candidate is a field used frequently in important queries. Indexes should follow actual query and access patterns—not the assumption that every field should have an index. For example, given:

```javascript
title: String,
category: String,
likes: Number,
description: String
```

an index like `promptSchema.index({ description: 1 })` may add little value if the application never queries by `description`.

#### Matching indexes to query patterns

```javascript
promptSchema.index({ category: 1 });

Prompt.find({ category: "Java" })
  .sort({ likes: -1 })
  .limit(10);
```

This query filters by `category`, sorts by `likes`, and limits to 10. The `{ category: 1 }` index can help the category filter, but it does not automatically optimize the `likes` sort. Indexed fields should match the query's access pattern.

#### Query planner, `IXSCAN`, and `COLLSCAN`

Having an index does not guarantee that MongoDB will use it. The query planner evaluates available execution plans and chooses the one it considers efficient:

```text
Index exists → query arrives → query planner → choose execution plan → may use the index
```

- `COLLSCAN`: MongoDB scans collection documents to find matching results.
- `IXSCAN`: MongoDB scans an index to locate relevant entries.

With only five documents, an index's performance benefit may be negligible even though it still has costs.

#### Index tradeoffs

```text
Faster reads
     ↕
Extra storage + write/maintenance overhead
```

Indexes improve reads but are not free. Inserts, updates, and deletes may require MongoDB to update relevant index entries. Create indexes strategically based on actual query patterns and performance needs.

```text
.index()              → creates/defines an index
_id                   → automatically indexed by MongoDB
1                     → ascending index order
-1                    → descending index order
Index                 → organized lookup structure / shortcut
Index stores          → indexed values + document references
Collection documents  → not reordered by the index
Query                 → does not create an index
Query planner         → decides whether to use an existing index
IXSCAN                → MongoDB is scanning an index
COLLSCAN              → MongoDB is scanning the collection
Good index field      → frequently used in important queries
Indexes are not free  → storage + write/maintenance cost
```

```text
Application Query
       ↓
Does a suitable index exist?
       ↓
Query Planner evaluates available plans
       ↓
If index is beneficial → MongoDB may use it
       ↓
IXSCAN
       ↓
Locate relevant documents efficiently
```

Don't create indexes just because a field exists. Create them based on application query patterns and performance needs.

Next: **Day 18 — Session 3: Compound Indexes** — putting multiple fields into one index and understanding why their order matters.
