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
