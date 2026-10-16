# Day 18 — Session 5: Index Trade-offs ⚖️

## Overview

Indexes can significantly improve read performance, but they are **not free**.

Every index introduces additional:
- Storage requirements
- Maintenance overhead
- Write overhead

Therefore, backend engineers should not blindly create indexes on every field.

The goal is to create **the right indexes for the application's actual query patterns and workload**.

---

# 1. Indexes Are Not Free

An index improves lookup performance by providing an additional data structure, but MongoDB must maintain that structure.

```text
More indexes
     ↓
More storage
     +
More maintenance
     +
More write overhead

For example:
promptSchema.index({ category: 1 });

MongoDB has to maintain this index as documents are inserted, updated, or deleted.
Key Point
Indexes optimize reads, but they can increase storage usage and write/maintenance overhead.

2. Indexing Should Be Based on Query Patterns
Do not create indexes simply because a field exists.
Instead, look at the application's actual access patterns.
For example, if the application frequently runs:
Prompt.find({ category: "Java" });

then an index on category may be useful.
But if description is rarely searched:
title
description
category
author
likes
createdAt
status

there may be no reason to index description.
Decision Process
Actual Query Patterns
        ↓
Which queries are important?
        ↓
Which fields are used?
        ↓
Would an index provide meaningful benefit?
        ↓
Create / keep / remove index

3. Read-Heavy vs Write-Heavy Workloads
Index strategy also depends on the workload.
Read-Heavy Application
If an application performs many reads, additional useful indexes may be worthwhile because faster queries can provide significant benefits.
Many Reads
    ↓
Indexes can improve lookup performance

Write-Heavy Application
If an application performs a huge number of writes, indexes must be chosen more carefully.
Every insert, update, or delete may require MongoDB to maintain relevant indexes.
Write
  ↓
Document changes
  ↓
Relevant indexes may also need updates
  ↓
Additional write work

Therefore:
Write-heavy applications should be more selective about which indexes they maintain.

4. Index Storage and Maintenance Cost
Suppose a collection has multiple indexes:
Documents
   +
category index
   +
likes index
   +
author index
   +
createdAt index
   +
status index

Every additional index can introduce:
- Storage cost
- Index maintenance
- Additional work during writes
Therefore:
More indexes do not automatically mean better performance.

5. Selectivity
Selectivity describes how effectively a query narrows down the number of matching documents.
High Selectivity
A query matches a small portion of the collection.
Example:
10 million documents

userId = "U123"
→ 20 matching documents

This is highly selective.
Low Selectivity
A query matches a large portion of the collection.
Example:
10 million documents

status = "active"
→ 9.5 million matching documents

This is low selectivity.
Mental Model
High Selectivity
      ↓
Small matching set
      ↓
Index may provide significant benefit

Low Selectivity
      ↓
Large matching set
      ↓
Index may provide limited benefit

Important
Low selectivity does not automatically mean an index is useless.
Other factors must also be considered:
- Query frequency
- Overall workload
- Execution plan
- Number of documents examined
- Other queries that may benefit from the same index
6. Avoid Redundant Indexes
Suppose we already have:
promptSchema.index({
  category: 1,
  likes: -1
});

We may not need another:
promptSchema.index({
  category: 1
});

because category is the leftmost field of the compound index.
The compound index can support category-based queries because of the leftmost-prefix rule.
Compound Index
{ category: 1, likes: -1 }

        ↓

category                ✅
category + likes        ✅
likes only              ❌ generally

Therefore, before creating another index, always ask:
"Do I already have an existing index that can support this query?"

7. Query Pattern + Existing Index
Consider:
Prompt.find({ category: "Java" })
  .sort({ likes: -1 })
  .limit(10);

With:
promptSchema.index({
  category: 1,
  likes: -1
});

the index matches the query pattern:
category → filter
likes    → sort

So creating additional separate indexes should not be done automatically.
The existing compound index may already provide the required support.
8. Real-World Index Decision
When deciding whether to create, keep, or remove an index, consider:
Query Pattern
      ↓
Query Frequency
      ↓
Selectivity
      ↓
Read vs Write Workload
      ↓
Existing Indexes
      ↓
Execution Plan / Performance
      ↓
Actual Benefit

The goal is not to maximize the number of indexes.
The goal is to maximize useful performance for the application's workload.
9. Example — Write-Heavy Application
Suppose:
10 million documents
100,000 writes/minute

and we have:
{ category: 1 }

but queries filtering by category are extremely rare.
Keeping this index may not be worthwhile if its read benefit is small compared with the ongoing maintenance and write overhead.
Before removing it in production, however, we should verify its usage and performance rather than deleting it blindly.
10. Index Decision Mental Model
             Query Pattern
                   ↓
           Query Frequency
                   ↓
              Selectivity
                   ↓
          Workload Analysis
          /               \
     Read-heavy        Write-heavy
          \               /
           ↓             ↓
       Measure actual benefit
                   ↓
          Check existing indexes
                   ↓
        Check execution plans
                   ↓
       Create / Keep / Remove

Key Takeaways
- Indexes improve read performance but have costs.
- Indexes require additional storage.
- Indexes can add write and maintenance overhead.
- Do not index every field blindly.
- Indexes should be designed around actual query patterns.
- Read-heavy applications may benefit from more useful indexes.
- Write-heavy applications require more careful index selection.
- Selectivity describes how much a query narrows the result set.
- High selectivity generally means fewer matching documents.
- Low selectivity does not automatically mean an index is useless.
- Always check whether an existing index already supports a query.
- Compound indexes can make separate single-field indexes unnecessary.
- The leftmost-prefix rule is important when evaluating compound indexes.
- Use execution plans and real workload measurements before making production index decisions.
🎯 Core Principle
Good indexing is not about having more indexes. It is about having the right indexes for the workload.

