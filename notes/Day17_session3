# Day 17 — Session 3: Query Practice & Interview Revision

## Overview

Session 3 focused on applying MongoDB/Mongoose querying concepts through practical PromptPulse scenarios and revising important interview concepts from Day 16 and Day 17.

The main goal was to move from understanding individual methods to **building complete backend queries from requirements**.

---

# 1. Combining Multiple Filters

Multiple conditions can be placed inside the same query object.

```javascript
Prompt.find({
  category: "Java",
  likes: { $gte: 100 }
});
```

This means:

```text
category = Java
AND
likes >= 100
```

For simple conditions on different fields, an explicit `$and` is usually not required.

---

# 2. `$or` Structure

`$or` is used when at least one condition should match.

```javascript
Prompt.find({
  $or: [
    { category: "Java" },
    { likes: { $gt: 500 } }
  ]
});
```

Important:

> `$or` takes an **array of condition objects**.

---

# 3. `$in` and `$nin`

### `$in`

Used when a field can have one of several specified values.

```javascript
Prompt.find({
  category: {
    $in: ["Java", "Python", "JavaScript"]
  }
});
```

### `$nin`

Used when a field should not match any of the specified values.

```javascript
Prompt.find({
  category: {
    $nin: ["Java", "Python"]
  }
});
```

---

# 4. Pagination Practice

For:

```text
page = 3
limit = 10
```

The skip value is:

```text
skip = (page - 1) × limit
     = (3 - 1) × 10
     = 20
```

Query:

```javascript
Prompt.find()
  .skip(20)
  .limit(10);
```

For page 2 with 10 items per page:

```javascript
.skip(10)
.limit(10)
```

---

# 5. Complete Query Example

Requirement:

> Get Java prompts with at least 100 likes, sort by highest likes, and return 10 prompts from page 2.

```javascript
const prompts = await Prompt.find({
  category: "Java",
  likes: { $gte: 100 }
})
  .sort({ likes: -1 })
  .skip(10)
  .limit(10);
```

Breakdown:

```text
find()   → Java + at least 100 likes
sort()   → highest likes first
skip()   → skip page 1
limit()  → return maximum 10
```

---

# 6. Projection Practice

### Include fields

```javascript
const users = await User.find()
  .select("name email");
```

Returns the selected fields.

### Exclude fields

```javascript
const users = await User.find()
  .select("-password -email");
```

Returns everything except `password` and `email`.

### Important

`.select()` **does not modify or delete data from MongoDB**.

It only controls which fields are included in the query result.

---

# 7. `$` — MongoDB Operators

The `$` prefix is used for MongoDB query operators.

Examples:

```text
$gt
$lt
$gte
$lte
$eq
$ne
$in
$nin
$or
$and
```

Example:

```javascript
{
  likes: { $gte: 100 }
}
```

Here:

```text
likes → field
$gte  → operator
100   → value
```

`likes` itself is not an operator.

Therefore:

```javascript
.sort({ likes: -1 })
```

is correct.

Not:

```javascript
.sort({ $likes: -1 })
```

---

# 8. Important `findByIdAndUpdate()` Interview Concept

By default:

```javascript
const prompt = await Prompt.findByIdAndUpdate(
  id,
  { likes: 200 }
);
```

updates the document in MongoDB but returns the **original document**.

To receive the updated document:

```javascript
const prompt = await Prompt.findByIdAndUpdate(
  id,
  { likes: 200 },
  { new: true }
);
```

### Important distinction

`new: true`:

> Does **not** make the update happen.

It tells Mongoose:

> Return the **updated document** instead of the original document.

---

# 9. Common Syntax Mistakes Identified

During practice, the main mistakes were mostly JavaScript/Mongoose syntax rather than misunderstanding the concepts.

### `find()`

Correct:

```javascript
Prompt.find({
  category: "Java",
  likes: { $gte: 100 }
});
```

### `sort()`

Correct:

```javascript
.sort({ likes: -1 })
```

### `select()`

Correct:

```javascript
.select("title likes")
```

or:

```javascript
.select("-password -email")
```

### `$or`

Correct structure:

```javascript
{
  $or: [
    { category: "Java" },
    { likes: { $gt: 500 } }
  ]
}
```

---

# 10. Key Interview Takeaways

### "At least"

```text
at least 100 → $gte: 100
```

Because 100 is included.

### "More than"

```text
more than 100 → $gt: 100
```

100 is not included.

### "At most"

```text
at most 100 → $lte: 100
```

### "Less than"

```text
less than 100 → $lt: 100
```

### AND

Multiple field conditions in the same query object represent an AND condition.

### OR

Use `$or` when at least one condition should match.

### Projection

```text
select("field1 field2")  → include selected fields
select("-field1")        → exclude field
```

---

# Session 3 Summary

The main lesson from Session 3 was:

> **Don't just memorize Mongoose methods. Learn to translate a backend requirement into a query.**

For example:

```text
"Java prompts"
        ↓
category: "Java"

"at least 100 likes"
        ↓
likes: { $gte: 100 }

"highest likes first"
        ↓
sort({ likes: -1 })

"page 2, 10 per page"
        ↓
skip(10).limit(10)

"only title and likes"
        ↓
select("title likes")
```

This ability to break an English requirement into individual query operations is important for backend interviews and real API development.
