# Day 18 — Session 1: MongoDB Indexing Fundamentals

## Overview

This session focused on understanding **why MongoDB queries can become slow** and how **database indexes** help improve query performance.

The goal was to understand the concept and trade-offs of indexing before learning how to create indexes.

---

# 1. Why Queries Can Become Slow

Consider a PromptPulse collection with:

```text
10 documents
```

Searching through the collection is inexpensive.

But as the application grows:

```text
10
↓
10,000
↓
1,000,000
↓
10,000,000 documents
```

A query that searches for a particular field value can become expensive if MongoDB has to examine a large number of documents.

For example:

```javascript
Prompt.find({
  category: "Java"
});
```

If there is no suitable index, MongoDB may need to examine documents throughout the collection to determine which ones match.

---

# 2. Collection Scan — COLLSCAN

When MongoDB scans documents in the collection to find matching documents, this is called a:

> **Collection Scan (`COLLSCAN`)**

Conceptually:

```text
Document 1 → category: Python → ❌
Document 2 → category: Java   → ✅
Document 3 → category: Go     → ❌
Document 4 → category: Java   → ✅
...
```

The important idea:

> Without a suitable index, MongoDB may need to scan a large portion of the collection to find matching documents.

---

# 3. What Is a Database Index?

An **index** is an additional data structure that helps MongoDB **locate relevant documents more efficiently**.

A useful analogy is the index of a book.

Without an index:

```text
Search → potentially go through many pages
```

With an index:

```text
Search term
   ↓
Index
   ↓
Relevant page locations
```

Similarly, a database index provides an organized lookup structure that can help MongoDB locate documents matching a query.

---

# 4. Field ≠ Index

An important distinction:

Having a field in a document does **not** automatically mean that MongoDB has an index for that field.

For example:

```javascript
{
  category: "Java"
}
```

means the document contains a `category` field.

It does not mean that MongoDB automatically has a lookup structure that allows it to instantly find every document where:

```text
category = "Java"
```

An index is a separate structure created to support efficient lookups.

---

# 5. Index Conceptual Example

Suppose PromptPulse contains:

```text
Document 1 → Python
Document 2 → Java
Document 3 → Go
Document 4 → Java
Document 5 → Python
```

Without an index, MongoDB may examine documents one by one.

Conceptually, an index on `category` can provide an organized structure such as:

```text
Java   → Document 2, Document 4
Go     → Document 3
Python → Document 1, Document 5
```

The index helps MongoDB find the relevant **index entries/locations** and then locate the corresponding documents.

### Important

The index should be thought of as a **lookup structure**, not as a second copy of the complete documents.

---

# 6. Index Scan — IXSCAN

When MongoDB uses an index while executing a query, the execution plan can contain:

> **Index Scan (`IXSCAN`)**

Conceptually:

```text
Query
  ↓
Index
  ↓
Relevant index entries
  ↓
Corresponding documents
```

So:

```text
COLLSCAN → Collection Scan
IXSCAN   → Index Scan
```

These terms become particularly useful when analyzing query execution later.

---

# 7. Indexes Are Not Free

Indexes can improve read performance, but they come with costs.

Indexes require:

* Additional storage
* Maintenance when data changes
* Additional work during write operations

For example, when documents are:

```text
INSERTED
UPDATED
DELETED
```

the relevant indexes may also need to be maintained.

Therefore, creating an index on every field is not automatically a good idea.

---

# 8. Indexes Should Be Used Strategically

The goal is not:

> "Create an index on everything."

Instead, we should consider:

> **Which queries actually need to be fast?**

Indexes should be created based on application **query patterns and access patterns**.

For a small collection, the performance difference may be negligible.

For a large collection with frequently executed queries, a suitable index can be much more valuable.

---

# 9. Important Subtlety

Having an index does **not** guarantee that MongoDB will always use it.

MongoDB's query planner determines whether using an available index is beneficial for a particular query.

Therefore:

```text
Index exists
      ≠
Index will always be used
```

This is something we will investigate later using query execution analysis.

---

# Key Takeaways

```text
COLLSCAN → MongoDB scans the collection to find matches

IXSCAN → MongoDB uses an index while executing the query

Index → additional organized lookup structure

Field ≠ Index

Indexes can improve reads
        +
Indexes consume storage
        +
Indexes add write/maintenance overhead
```

### Core Mental Model

```text
WITHOUT SUITABLE INDEX

Query
  ↓
COLLSCAN
  ↓
Potentially examine many documents
  ↓
Find matching documents


WITH SUITABLE INDEX

Query
  ↓
Index
  ↓
Locate relevant index entries
  ↓
Find corresponding documents
```

### Final Principle

> **An index is a performance optimization that helps MongoDB find data efficiently, but it comes with storage and maintenance costs. Therefore, indexes should be created based on real query patterns rather than blindly indexing every field.**
