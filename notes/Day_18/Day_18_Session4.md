# Day 18 — Session 4: `explain()` & Query Execution

## 🔍 Phase 1–3: Understanding Query Execution

### Why `explain()`?

Creating an index does not guarantee that MongoDB will use it. The **query planner** decides which execution plan is appropriate for a query.

`explain()` helps us inspect **how MongoDB plans and/or executes a query**.

```javascript
Prompt.find({ category: "Java" }).explain("executionStats");
```

Instead of simply getting the documents, we can inspect:

- Whether an index was used
- Which execution plan was chosen
- How many documents were examined
- How many index keys were examined
- How many documents were returned

```text
Query
  ↓
Query Planner
  ↓
Execution Plan
  ↓
explain()
  ↓
Understand HOW MongoDB found the results
```

### `explain()` verbosity modes

```javascript
Prompt.find({ category: "Java" }).explain("queryPlanner");
```

Focuses on the planned execution strategy.

```javascript
Prompt.find({ category: "Java" }).explain("executionStats");
```

Provides actual execution statistics, making it particularly useful for analyzing query performance.

```javascript
Prompt.find({ category: "Java" }).explain("allPlansExecution");
```

Provides information about the candidate execution plans considered by MongoDB.

```text
queryPlanner       → planned approach
executionStats     → actual execution + statistics
allPlansExecution  → candidate plans
```

### Key execution statistics

- `nReturned`: number of documents ultimately returned by the query.
- `totalKeysExamined`: number of index keys examined during query execution.
- `totalDocsExamined`: number of documents examined during query execution.

Example:

```text
nReturned: 10
totalKeysExamined: 10
totalDocsExamined: 10
```

This generally indicates that MongoDB did relatively little examination work for the returned results. Compare it with:

```text
nReturned: 10
totalKeysExamined: 500000
totalDocsExamined: 500000
```

MongoDB examined a large amount of data to return only 10 documents, which is suspicious and worth investigating.

```text
nReturned
    ↓
Documents returned

totalKeysExamined
    ↓
Index keys examined

totalDocsExamined
    ↓
Documents examined
```

### `COLLSCAN` and `IXSCAN`

`COLLSCAN` means MongoDB scans documents in the collection to find matching documents.

```text
Collection
 ↓
Doc 1 → check
Doc 2 → check
Doc 3 → check
Doc 4 → check
...
```

A `COLLSCAN` does not necessarily mean an index existed and MongoDB ignored it. It simply means the chosen execution stage is scanning the collection.

`IXSCAN` means MongoDB is scanning an index to find relevant index entries.

```text
Query
  ↓
Index
  ↓
Relevant index entries
  ↓
Document references
```

```text
COLLSCAN → scans collection

IXSCAN   → scans index
```

### `FETCH`

The query planner decides which execution plan to use. Sometimes MongoDB uses:

```text
IXSCAN
   ↓
FETCH
```

`IXSCAN` finds relevant index entries and document references. `FETCH` then retrieves the actual documents from the collection using those references.

```text
Query
  ↓
IXSCAN
  ↓
Relevant index entries
  ↓
Document references
  ↓
FETCH
  ↓
Actual documents
```

### Covered queries

A query can sometimes be answered entirely using the index, without fetching the actual documents. For example:

```javascript
promptSchema.index({
  category: 1,
  likes: -1
});
```

```javascript
Prompt.find({ category: "Java" }).select("category likes");
```

The index already contains `category` and `likes`, which are the fields needed by the query. Therefore, MongoDB may be able to use only the index.

```text
Query
  ↓
IXSCAN
  ↓
All required fields available in index
  ↓
No FETCH needed
```

This is called a **covered query**.

With `IXSCAN → FETCH`, MongoDB scans the index and then retrieves the actual documents. With only `IXSCAN`, the required information can potentially be obtained directly from the index, avoiding the additional document-fetch step.

### Summary

- `explain()` helps us understand how MongoDB plans and/or executes a query and inspect execution statistics.
- `COLLSCAN` scans documents in the collection to find matching documents.
- `IXSCAN` scans an index to find relevant index entries.
- `FETCH` retrieves actual documents using references found through the index.
- `nReturned` is the number of documents returned by the query.
- `totalKeysExamined` is the number of index keys examined during execution.
- `totalDocsExamined` is the number of documents examined during execution.
- A **covered query** can potentially be answered entirely from an index without fetching the actual documents.

```text
Query
  ↓
Query Planner
  ↓
Execution Plan
  ↓
 ┌─────────────────┐
 │                 │
COLLSCAN       IXSCAN
 │                 │
Collection       Index
                  ↓
               FETCH
                  ↓
           Actual Documents
```

If all required information is already available in the index:

```text
Query
  ↓
IXSCAN
  ↓
Required information
  ↓
No FETCH needed
```

`explain()` allows us to move from assuming a query is efficient to actually inspecting how MongoDB executed it.

> Phase 1 → Phase 3 documented and locked. 🔒
>
> Next time: **Phase 4: Indexed vs Non-Indexed Queries**.
