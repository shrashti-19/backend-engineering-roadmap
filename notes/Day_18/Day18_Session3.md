# Day 18 — Session 3: Compound Indexes

## 📌 Overview

A **compound index** is an index built using **multiple fields**.

Unlike having two separate single-field indexes, a compound index creates **one index structure containing multiple fields in a specific order**.

Example:

```javascript
promptSchema.index({ category: 1, likes: -1 });
```

This creates a compound index where:

1. `category` is the first field.
2. `likes` is the second field.

The order of fields is important because MongoDB organizes the index hierarchically based on that order.

## How It Is Organized

A compound index contains multiple fields in a single index.

```javascript
promptSchema.index({ category: 1, likes: -1 });
```

Conceptually:

```text
category
   ↓
likes
   ↓
document reference
```

For example:

```text
Java
 ├── 900 → Document
 ├── 700 → Document
 └── 300 → Document

Node
 ├── 800 → Document
 └── 400 → Document
```

MongoDB organizes the index according to the field order.

## Design Around Query Patterns

Compound indexes are useful when queries commonly use multiple fields together:

```javascript
Prompt.find({ category: "Java" })
  .sort({ likes: -1 })
  .limit(10);
```

This query filters by `category`, sorts by `likes`, and returns the top 10 results. A suitable index is:

```javascript
promptSchema.index({ category: 1, likes: -1 });
```

The index follows the query's access pattern:

```text
Query
  ↓
Filter by category
  ↓
Narrow down matching documents
  ↓
Work with likes ordering
  ↓
Return relevant results
```

The purpose is not merely to index multiple fields at once. Design the index around a common multi-field query pattern.

These indexes are **not** equivalent:

```javascript
{ category: 1, likes: -1 }
{ likes: -1, category: 1 }
```

The first is organized roughly as `category → likes`; the second is `likes → category`. For this query:

```javascript
Prompt.find({ category: "Java" })
  .sort({ likes: -1 });
```

`{ category: 1, likes: -1 }` is the better fit because it matches `category → likes`.

## Leftmost Prefix Rule

Given this index:

```javascript
{ category: 1, likes: -1 }
```

`category` is the leftmost field. The index can effectively support these prefixes:

```text
Index: A → B → C

A              ✅
A + B          ✅
A + B + C      ✅

B only         ❌  (not an effective leftmost-prefix match)
C only         ❌  (not an effective leftmost-prefix match)
```

For `{ category: 1, likes: -1 }`, these are good candidates:

```javascript
Prompt.find({ category: "Java" });

Prompt.find({
  category: "Java",
  likes: { $gte: 100 }
});
```

A query using only `likes` does not match the leftmost prefix:

```javascript
Prompt.find({ likes: { $gte: 100 } });
```

Think of the index as:

```text
category
   ↓
likes
   ↓
document
```

MongoDB can first narrow down through `category`, then work within that portion using `likes`. If it only receives `{ likes: { $gte: 100 } }`, it does not have the first field needed to narrow down this hierarchical structure.

The Leftmost Prefix Rule is about which fields can be used effectively **from the beginning** of a compound index. It is not about the direction MongoDB is allowed to scan the index.

## Sort Directions and Reverse Scans

For this index:

```javascript
{ category: 1, likes: -1 }
```

`1` means ascending and `-1` means descending, so the index order is:

```text
category ↑
likes    ↓
```

MongoDB can traverse an index in reverse. Reversing this compound index gives:

```text
category ↓
likes    ↑
```

For a compound index, reverse traversal reverses the direction of **all** indexed fields together. A difference between `1` and `-1` therefore does not automatically make the index useless; compatibility depends on the overall sort pattern.

## Example: Filter, Sort, Limit

```javascript
promptSchema.index({ category: 1, likes: -1 });

Prompt.find({ category: "Java" })
  .sort({ likes: -1 })
  .limit(10);
```

The query pattern is:

```text
Filter → Sort → Limit
category → likes → 10 results
```

The index follows the same pattern (`category → likes`), making it a strong candidate.

## Single-Field vs Compound Index

```javascript
promptSchema.index({ category: 1 });
```

This focuses on one field and can help with:

```javascript
Prompt.find({ category: "Java" });
```

```javascript
promptSchema.index({
  category: 1,
  likes: -1
});
```

This is designed for a multi-field access pattern:

```javascript
Prompt.find({ category: "Java" })
  .sort({ likes: -1 });
```

```text
Single-field index: category
Compound index:     category → likes
```

## Query Planner

Having a compound index does not guarantee MongoDB will always use it. MongoDB's query planner evaluates available plans and chooses what it considers appropriate for the query.

## Filing-System Analogy

For `{ category: 1, likes: -1 }`, think of a hierarchical filing system:

```text
Category
   ↓
  Java
   ├── 900 likes
   ├── 700 likes
   └── 300 likes

  Node
   ├── 800 likes
   └── 400 likes
```

Knowing `category = Java` lets MongoDB enter the Java section and then work with likes. Knowing only `likes = 700` does not provide the first level of the hierarchy.

## Key Takeaways

- A compound index is one index structure containing multiple fields in a specific order.
- Field order changes how the index is organized and which query patterns it can efficiently support.
- `{ category: 1, likes: -1 }` is one compound index, not simply separate `{ category: 1 }` and `{ likes: -1 }` indexes.
- `{ category: 1, likes: -1 }` and `{ likes: -1, category: 1 }` are different indexes.
- A query using `category` matches the leftmost prefix; a query using only `likes` does not.
- MongoDB can traverse an index in reverse; the important issue is the overall ordering pattern.
- Design compound indexes around real query patterns, then verify actual query performance.

```text
Query Pattern
      ↓
Understand how data is accessed
      ↓
Choose fields
      ↓
Choose field order
      ↓
Create compound index
      ↓
Verify actual query performance
```

This documents **everything we've genuinely covered in Session 3 today**—nothing from the unfinished practice/interview portion is being sneakily added. 😌

Once you've saved it, we can continue from **Session 3 practice** exactly where we left off.
