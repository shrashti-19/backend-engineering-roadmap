# MongoDB Query Operations — Session 1

## Sorting Documents with `sort()`

`sort()` is used to arrange the documents returned by a query based on a particular field.

```javascript
Prompt.find().sort({ likes: -1 });
```

Here:

* `1` → ascending order (low → high)
* `-1` → descending order (high → low)

For example:

```javascript
Prompt.find().sort({ likes: -1 });
```

returns the prompts with the highest likes first.

### Important

`sort()` changes the **order of documents**, not the fields returned.

---

## Limiting Results with `limit()`

`limit()` is used when we want to restrict how many documents a query returns.

```javascript
Prompt.find().limit(5);
```

This means:

> Return at most 5 documents.

If only 3 documents exist, it returns 3. It does not create extra documents.

`limit(5)` means **maximum 5**, not exactly 5.

---

## Skipping Documents with `skip()`

`skip()` is used to ignore a certain number of documents from the query result.

```javascript
Prompt.find().skip(10);
```

This means:

> Skip the first 10 matching documents.

`skip()` does not delete anything from the database. It only affects what the current query returns.

---

## Combining `sort()`, `skip()` and `limit()`

These methods can be combined to create paginated results.

```javascript
Prompt.find()
  .sort({ likes: -1 })
  .skip(10)
  .limit(10);
```

This means:

1. Sort prompts by likes from highest to lowest.
2. Skip the first 10 prompts.
3. Return the next 10 prompts.

So this can be used to get **page 2**, when each page contains 10 prompts.

---

## Pagination Formula

To calculate how many documents to skip:

```text
skip = (page - 1) × limit
```

For example, if:

```text
page = 3
limit = 10
```

Then:

```text
skip = (3 - 1) × 10
     = 20
```

So page 3 would use:

```javascript
Prompt.find()
  .skip(20)
  .limit(10);
```

---

## Handling Fewer Documents

`limit()` returns **at most** the specified number of documents.

For example, if the database has 50 documents:

```javascript
Prompt.find()
  .skip(45)
  .limit(10);
```

After skipping 45 documents, only 5 remain, so the query returns **5 documents**, not 10.

If:

```javascript
Prompt.find()
  .skip(50)
  .limit(10);
```

all 50 documents are skipped, so the result is an empty array.

---

## Real Backend Example

Suppose PromptPulse needs to show the most-liked prompts, 10 at a time.

For page 2:

```javascript
const prompts = await Prompt.find()
  .sort({ likes: -1 })
  .skip(10)
  .limit(10);
```

This gives:

> Most-liked prompts → skip page 1 → return page 2.

---

## Key Takeaways

* `sort()` → controls the **order** of documents.
* `1` → ascending, `-1` → descending.
* `limit()` → controls the **maximum number** of documents returned.
* `skip()` → ignores a number of documents in the current query.
* `skip()` does **not** delete documents.
* `sort() + skip() + limit()` can be used for pagination.
* Pagination formula:

```text
skip = (page - 1) × limit
```
