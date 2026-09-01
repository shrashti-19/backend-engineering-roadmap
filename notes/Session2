# Day 17 — MongoDB Querying & Pagination

## Overview

In this session, I learned how to build more practical MongoDB queries using Mongoose.

The focus was on **filtering, sorting, pagination, projection, and combining multiple query operations**.

These concepts are important for building real backend APIs where clients need filtered, sorted, paginated, and limited data.

---

# 1. Sorting

Mongoose provides `.sort()` to control the order of returned documents.

### Descending

```javascript
Prompt.find().sort({ likes: -1 });
```

Returns documents from **highest likes to lowest likes**.

### Ascending

```javascript
Prompt.find().sort({ likes: 1 });
```

Returns documents from **lowest likes to highest likes**.

### Important

`sort()` changes the **order of documents**. It does not decide which fields are returned.

---

# 2. limit()

`.limit()` controls the maximum number of documents returned.

```javascript
Prompt.find().limit(10);
```

This returns **at most 10 documents**.

If only 5 matching documents exist, only 5 are returned.

> `limit(10)` does not guarantee that exactly 10 documents will be returned.

---

# 3. skip()

`.skip()` ignores a specified number of matching documents.

```javascript
Prompt.find().skip(10);
```

This skips the first 10 documents in the current query result.

It does **not delete** those documents.

---

# 4. Pagination

Pagination is used to divide a large set of results into smaller pages.

A common formula is:

```text
skip = (page - 1) × limit
```

For example:

```text
page = 3
limit = 10

skip = (3 - 1) × 10
     = 20
```

So page 3 skips the first 20 documents and returns the next 10.

### Example

```javascript
Prompt.find()
  .skip(20)
  .limit(10);
```

---

# 5. Combining sort(), skip() and limit()

Example:

```javascript
Prompt.find()
  .sort({ likes: -1 })
  .skip(10)
  .limit(10);
```

This means:

1. Sort prompts by likes, highest first.
2. Skip the first 10.
3. Return the next 10.

This can be used to implement pagination for a "most liked prompts" page.

---

# 6. Filtering with Comparison Operators

MongoDB provides operators for comparing field values.

| Operator | Meaning                  |
| -------- | ------------------------ |
| `$gt`    | Greater than             |
| `$lt`    | Less than                |
| `$gte`   | Greater than or equal to |
| `$lte`   | Less than or equal to    |
| `$eq`    | Equal to                 |
| `$ne`    | Not equal to             |

### `$gt`

```javascript
Prompt.find({
  likes: { $gt: 100 }
});
```

Finds prompts with **more than 100 likes**.

100 itself is not included.

### `$lt`

```javascript
Prompt.find({
  likes: { $lt: 100 }
});
```

Finds prompts with **less than 100 likes**.

### `$gte`

```javascript
Prompt.find({
  likes: { $gte: 100 }
});
```

Finds prompts with **100 or more likes**.

### `$lte`

```javascript
Prompt.find({
  likes: { $lte: 100 }
});
```

Finds prompts with **100 or fewer likes**.

### `$eq`

```javascript
Prompt.find({
  likes: { $eq: 100 }
});
```

Finds prompts with **exactly 100 likes**.

Simple equality can also be written as:

```javascript
Prompt.find({
  likes: 100
});
```

### `$ne`

```javascript
Prompt.find({
  likes: { $ne: 100 }
});
```

Finds prompts where likes are **not equal to 100**.

---

# 7. $in

`$in` is used when a field can match **one of several values**.

```javascript
Prompt.find({
  category: {
    $in: ["Java", "Python", "Go"]
  }
});
```

This returns prompts whose category is:

* Java
* Python
* Go

Think:

> "Is the value one of these?"

---

# 8. $nin

`$nin` means the value should **not be any of the specified values**.

```javascript
Prompt.find({
  category: {
    $nin: ["Java", "Python"]
  }
});
```

This excludes prompts whose category is Java or Python.

Think:

> "Is the value none of these?"

---

# 9. $and

`$and` means **all conditions must be true**.

```javascript
Prompt.find({
  $and: [
    { likes: { $gt: 100 } },
    { category: "Java" }
  ]
});
```

The prompt must:

* have more than 100 likes
* AND belong to the Java category

### Important

For simple conditions on different fields, explicit `$and` is usually unnecessary.

This:

```javascript
Prompt.find({
  category: "Java",
  likes: { $gt: 100 }
});
```

already means:

```text
category is Java
AND
likes > 100
```

---

# 10. $or

`$or` means **at least one condition must be true**.

```javascript
Prompt.find({
  $or: [
    { category: "Java" },
    { likes: { $gt: 500 } }
  ]
});
```

This returns prompts that are:

* Java prompts

**OR**

* prompts with more than 500 likes.

Only one condition needs to match.

---

# 11. Projection with select()

Projection controls **which fields are returned** from the matching documents.

### Include specific fields

```javascript
const users = await User.find()
  .select("name email");
```

This returns only the selected fields.

### Exclude a field

```javascript
const users = await User.find()
  .select("-password");
```

This excludes `password` from the returned result.

It does **not delete** the password from MongoDB.

### Exclude multiple fields

```javascript
User.find()
  .select("-password -email");
```

This means:

> Return everything except `password` and `email`.

### Important mental model

**Filtering** decides:

> Which documents do I want?

**Projection** decides:

> Which fields from those documents do I want?

---

# 12. Combining Everything

A real PromptPulse query can combine all these concepts.

Requirement:

> Find Java prompts with at least 100 likes, sort by highest likes, return the next 10 prompts, and only return their title and likes.

```javascript
const prompts = await Prompt.find({
  category: "Java",
  likes: { $gte: 100 }
})
  .sort({ likes: -1 })
  .limit(10)
  .select("title likes");
```

### Query breakdown

```text
find()
→ Java prompts with at least 100 likes

sort()
→ highest likes first

limit()
→ maximum 10 prompts

select()
→ return only title and likes
```

---

# 13. Pagination + Filtering Example

Suppose we want **page 2** with 10 prompts per page.

```javascript
const prompts = await Prompt.find({
  category: "Java",
  likes: { $gte: 100 }
})
  .sort({ likes: -1 })
  .skip(10)
  .limit(10)
  .select("title likes");
```

This means:

> Find Java prompts with at least 100 likes → sort by highest likes → skip the first 10 → return the next 10 → only send title and likes.

---

# 14. Important Interview Notes

### `$gt` vs `$gte`

```text
$gt  → strictly greater than
$gte → greater than or equal to
```

Therefore:

```text
"more than 100" → $gt: 100

"at least 100" → $gte: 100
```

### `$lt` vs `$lte`

```text
$lt  → strictly less than
$lte → less than or equal to
```

### `$in` vs `$nin`

```text
$in  → one of these values
$nin → none of these values
```

### `$and` vs `$or`

```text
$and → all conditions must match
$or  → at least one condition must match
```

### `select()` does not modify the database

It only controls the fields included in the query result.

---

# 15. Common Mistakes

### Mistake 1 — Using `$` with field names

Incorrect:

```javascript
.sort({ $likes: -1 })
```

Correct:

```javascript
.sort({ likes: -1 })
```

`$` is used for MongoDB operators such as `$gt`, `$gte`, `$in`, `$or`, etc.

---

### Mistake 2 — Putting filters outside find()

Incorrect:

```javascript
Prompt.find({ category: "Java" })
  .{ likes: { $gte: 100 } }
```

Correct:

```javascript
Prompt.find({
  category: "Java",
  likes: { $gte: 100 }
});
```

---

### Mistake 3 — Confusing select() with deleting fields

```javascript
.select("-password")
```

does **not** delete `password`.

It only prevents the field from being included in the returned result.

---

# Key Takeaways

```text
sort()   → controls order
skip()   → skips documents
limit()  → controls maximum results
find()   → filters documents
select() → controls returned fields

$gt      → >
$lt      → <
$gte     → >=
$lte     → <=
$eq      → =
$ne      → !=
$in      → one of
$nin     → none of
$and     → all conditions
$or      → at least one condition
```

These operations form the foundation for building practical backend endpoints such as:

* paginated feeds
* search results
* filtered dashboards
* most-liked content
* category-based listings
* user-specific queries
* API responses with only required fields

````

This is **Day 17 — Session 2 documentation**. Add it alongside Session 1, then your Day 17 documentation is complete. 🚀

For the GitHub commit, I'd use:

```bash
git add .
git commit -m "docs: add MongoDB querying and filtering"
git push
````

**Next session:** we'll move beyond basic querying into the next backend layer rather than endlessly squeezing MongoDB for knowledge 😂.
