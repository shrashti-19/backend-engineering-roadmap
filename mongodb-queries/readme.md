## In this chapter, we'll learn how a Node.js application communicates with MongoDB to retrieve, insert, update, and delete data using Mongoose query methods.
## What is MongoDB query?
A MongoDB query is used to retrieve documents from a collection based on specified conditions, using a JavaScript-like syntax.

## Why do we need MongoDB Queries?
- Without queries: we cannot interact with the database.
- Queries help to perform CRUD operations.
- They allow us to search for specific data instead of reading the entire databse.

Example:
- Find a user by email during login.
- Create an user during Signup.

## Common MongoDB Query Methods Covered
- find() - retrieves multiple documents.
- findOne() - retrives a single document matching the document.
- findById() - retrieves the document matching its _id.
- create() - inserts a new document into collection.

## Asynchronous Nature of MongoDB Queries
- Database operations take time.
- Node.js doesn't stop the entire program while waiting.
- MongoDB queries run asynchronously.

## Why do MongoDB Queries return Promises?
Most Mongoose operations like find(), findOne(), findById(), create() are asynchronous data operations and return Promises. We use await so that JS waits for the DB creating the document before moving to the next line of code and gives us the resolved value of
the Promise.

## Why do we use await with MongoDB Queries?
We use await with MongoDB queries because awaits waits for the promise to be resolved.
Without awaits, we'll get a Promise object instead of the actual result.

## Example:
const user = User.findById(id);
console.log(user.name)

Output: undefined, error, user is still a promise

const user = await User.findById(id);
console.log(user.name)

Output: name of the user


## Difference between Array and Object in Query Results
| Method       | Returns          |
| ------------ | ---------------- |
| `find()`     | Array of Objects |
| `findOne()`  | Object or `null` |
| `findById()` | Object or `null` |
| `create()`   | Object           |

## Summary

- MongoDB queries allow us to interact with the database by retrieving and inserting data.
- Mongoose provides query methods such as `find()`, `findOne()`, `findById()`, and `create()` to perform database operations.
- MongoDB queries are asynchronous and return Promises because database operations take time to complete.
- We use `await` to wait for the Promise to resolve and get the actual result.
- Depending on the query method, the returned result can be an array of objects or a single object.
- Understanding these fundamentals is essential before learning advanced query methods such as updating, deleting, filtering, sorting, and pagination.
