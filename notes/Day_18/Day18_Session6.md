# Day 18 — Session 6: PromptPulse Performance Challenge 🚀

## Overview

Applied MongoDB indexing concepts to real-world PromptPulse query scenarios.

The focus was on designing compound indexes, understanding query execution, and making performance optimization decisions.

---

## 1. Compound Index Design

For queries involving multiple filter and sort fields, consider a compound index.

Example:

```javascript
Prompt.find({ category: "Java" })
  .sort({ likes: -1 })
  .limit(10);
```

Index:

```javascript
promptSchema.index({
  category: 1,
  likes: -1
});
```

- `category` → Filtering
- `likes` → Descending sort

## 2. Multiple Filter Conditions

Example:

```javascript
Prompt.find({
  category: "Java",
  isDeleted: false
})
.sort({ likes: -1 })
.limit(10);
```

Possible compound index:

```javascript
promptSchema.index({
  category: 1,
  isDeleted: 1,
  likes: -1
});
```

Both filter fields use equality conditions, followed by the sort field.

## 3. ESR vs ERS

**ESR — Equality → Sort → Range**

```javascript
promptSchema.index({
  author: 1,
  createdAt: -1,
  likes: 1
});
```

- Prioritizes supporting the requested sort.
- Can avoid an additional in-memory sort.

**ERS — Equality → Range → Sort**

```javascript
promptSchema.index({
  author: 1,
  likes: 1,
  createdAt: -1
});
```

- Can reduce index entries examined when the range is highly selective.
- May require an additional sort.

ESR is a guideline, not an absolute rule. Actual performance should be verified using `explain("executionStats")`.

## 4. Leftmost-Prefix Rule

For the compound index:

```javascript
{ category: 1, likes: -1 }
```

- `category` → Supported
- `category + likes` → Supported
- `likes` only → Generally not efficiently supported

Compound indexes are organized around their leading fields.

## 5. Redundant Indexes

Before creating a new index, check whether an existing compound index already supports the query.

Example:

```javascript
{ author: 1 }
{ author: 1, createdAt: -1 }
```

The compound index can generally support queries using only `author` because it is the leftmost field.

A separate `{ author: 1 }` index may therefore be redundant.

## 6. Covered Queries

A covered query can return results directly from an index without retrieving actual documents.

Example:

```javascript
promptSchema.index({
  category: 1,
  likes: -1
});

Prompt.find({ category: "Java" })
  .select("category likes");
```

If all required fields are available in the index, MongoDB may avoid the `FETCH` stage.

## 7. Query Performance Analysis

Important execution statistics:

| Metric | Meaning |
|---|---|
| `nReturned` | Number of documents returned |
| `totalKeysExamined` | Number of index keys examined |
| `totalDocsExamined` | Number of documents examined |

Important execution stages:

- `IXSCAN` → Scans index entries.
- `FETCH` → Retrieves actual documents.
- `COLLSCAN` → Scans collection documents.

**Important:** Using an index does not automatically guarantee an efficient query.

A large difference between `totalKeysExamined` and `nReturned` can indicate that the query is examining many unnecessary index entries.

## 8. Production Index Decisions

When optimizing indexes, consider:

- Query patterns and frequency.
- Selectivity.
- Existing indexes.
- Read vs write workload.
- Storage and maintenance costs.
- Actual execution statistics.

Do not remove useful indexes blindly just because write overhead is increasing. Measure the read-performance benefit against the write cost before deciding.

---

## Key Takeaways

- Design compound indexes around actual query patterns.
- Apply the ESR guideline when choosing field order.
- Understand when ERS may be more suitable.
- Follow the leftmost-prefix rule.
- Avoid redundant indexes.
- Understand covered queries and the `FETCH` stage.
- Evaluate `nReturned`, `totalKeysExamined`, and `totalDocsExamined`.
- Verify optimization decisions using `explain("executionStats")`.

## Core Principle

> A good backend engineer doesn't just create indexes. They design indexes around real query patterns, analyze execution plans, and measure whether those indexes actually improve performance.