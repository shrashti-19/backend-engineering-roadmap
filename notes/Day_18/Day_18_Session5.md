
## 🔍 Phase 4–7: Query Performance Analysis & Interview Practice

### Indexed vs Non-Indexed Queries

`explain("executionStats")` can be used to compare how much work MongoDB performs for different execution plans.

Example:
```text
Without suitable index:

COLLSCAN
nReturned: 100
totalDocsExamined: 1,000,000
```

With a suitable index:
```yaml
IXSCAN → FETCH
nReturned: 100
totalKeysExamined: 100
totalDocsExamined: 100
```

The second execution performs significantly less examination work to return the same number of documents.

### Important Principle

> Don't assume an index improves performance. Use `explain("executionStats")` to verify the actual execution behavior.

---

## 🕵️ Query Performance Diagnosis

### `IXSCAN` does not automatically mean perfect optimization.

Example:
```yaml
IXSCAN
nReturned: 100
totalKeysExamined: 5,000
totalDocsExamined: 100
```

The index is being used, but MongoDB examined many more index keys than documents returned.

This means the query may need further investigation.

Possible factors include:

- Query selectivity
- Index design
- Number of matching index entries
- Overall query pattern

---

### `COLLSCAN` does not automatically mean a bad query.

Example:
```yaml
COLLSCAN
nReturned: 100
totalDocsExamined: 100
```

If the collection contains only 100 documents, examining all 100 may be perfectly reasonable.

Therefore, execution plans should be evaluated using context rather than blindly following:
```ini
COLLSCAN = bad
IXSCAN = good
```

---

# 📊 Comparing Execution Plans

Consider a collection with 1 million documents.

### Plan A
```yaml
COLLSCAN
nReturned: 100
totalDocsExamined: 1,000,000
```

### Plan B
```yaml
IXSCAN → FETCH
nReturned: 100
totalKeysExamined: 100
totalDocsExamined: 100
```

Plan B performs significantly less work to return the same number of documents.

### Mental Model
```perl
Execution Plan
      ↓
Did MongoDB use an index?
      ↓
IXSCAN / COLLSCAN
      ↓
How much work was performed?
      ↓
totalKeysExamined
totalDocsExamined
      ↓
How many documents were actually returned?
      ↓
nReturned
```

---

# 🎯 Interview Takeaways

### 1. Does `IXSCAN` always mean the query is optimized?

> No. An index may be used while MongoDB still examines a large number of index keys. Execution statistics must be analyzed.

### 2. Does `COLLSCAN` always mean poor performance?

> No. For a small collection, scanning all documents may be reasonable.

### 3. How can we verify whether an index is helping?

> Use `explain("executionStats")` and analyze the execution stage along with `nReturned`, `totalKeysExamined`, and `totalDocsExamined`.

### 4. What does a large `totalDocsExamined` compared with `nReturned` indicate?

> MongoDB is examining significantly more documents than it ultimately returns, which can indicate inefficient query execution and should be investigated.

### 5. Why is `explain()` important?

> It allows us to measure and understand how MongoDB actually executes a query instead of assuming that an index automatically improves performance.

---

# 🔑 Session 4 Core Principle

> **Query performance should be measured, not assumed. `explain("executionStats")` helps us understand the execution plan and the amount of work MongoDB performs to return the requested data.**

```graphql
Query
  ↓
Query Planner
  ↓
Execution Plan
  ↓
COLLSCAN / IXSCAN
  ↓
Execution Statistics
  ↓
Analyze Performance
  ↓
Optimize if Necessary
```

### Session 4 Status

✅ `explain()` basics  
✅ Execution statistics  
✅ `COLLSCAN`  
✅ `IXSCAN`  
✅ `FETCH`  
✅ Covered queries  
✅ Indexed vs non-indexed comparison  
✅ Query performance diagnosis  
✅ Interview questions

**Session 4 = officially COMPLETE 🔒**

Next up: **Day 18 — Session 5: Index Trade-offs** ⚖️