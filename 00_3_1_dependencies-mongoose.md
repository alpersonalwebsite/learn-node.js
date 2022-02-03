## Mongoose
Object Data Modeling (ODM) library for MongoDB and Node.js.

Install mongoose: `npm install mongoose`

## Connecting to MongoDB

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/my-app')
  .then(() => console.log('Connected to MongoDB!'))
  .catch(err => console.error(err));
```

Output:
```
Connected to MongoDB!
```

For the following steps, you will need to have a database and collection created.
Using `Compass`, create a new database (`my-app`) and a new collection (`users`).

## Initial terminology

* `Schema` will define the shape of the documents within a MongoDB collection.
* `Model` is a compiled version of the schema. We can use the resulting class to initialize objects based on this class.
* `Collection` is similar to tables in relational databases. Can hold multiple JSOn documents.
* `Documents` is similar to a row in relational databases. 

**IMPORTANT:** MongoDB is schema-less. A schema is a Mongoose concept.

## Creating a Schema

The following are all the valid SchemaTypes in Mongoose. Mongoose plugins can also add custom SchemaTypes like int32.
More info: https://mongoosejs.com/docs/schematypes.html

* String
* Number
* Date
* Buffer
* Boolean
* Mixed
* ObjectId
* Array
* Decimal128
* Map
* Schema

```js
const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  hobbies: [String]
})
```

## Creating a Model

```js
// User is the singular name of the collection Users
const User = mongoose('User', userSchema);

// With the User class we can create objects 
const user = new User({
  name: 'Peter',
  age: 33,
  hobbies: [ 'reading', 'writing' ]
});
```

## Creating a Document

```js
async function createUser() {
  // With the User class we can create objects 
  const user = new User({
    name: 'Peter',
    age: 33,
    hobbies: ['reading', 'writing']
  });

  const result = await user.save();
  console.log(result);
}

createUser();
```

Output:

```
{
  name: 'Peter',
  age: 33,
  hobbies: [ 'reading', 'writing' ],
  _id: new ObjectId("61fc547ff914288905bc801f"),
  __v: 0
}
```

## Querying or retrieving Documents

### Retrieving ALL documents in a collection

```js
async function getUsers() {
  const result = await User.find();
  console.log(result);
}

getUsers();
```

Output:
```
[
  {
    _id: new ObjectId("61fc547ff914288905bc801f"),
    name: 'Peter',
    age: 33,
    hobbies: [ 'reading', 'writing' ],
    __v: 0
  },
  {
    _id: new ObjectId("61fc5507e0b044d7c5b88d94"),
    name: 'Peter',
    age: 33,
    hobbies: [ 'reading', 'writing' ],
    __v: 0
  },
  {
    _id: new ObjectId("61fc550c57ddee4b3f715958"),
    name: 'Peter',
    age: 33,
    hobbies: [ 'reading', 'writing' ],
    __v: 0
  }
]
```

### Retrieving some documents
We can filter documents based on a specific criteria. 

```js
async function getUsers() {
  const result = await User.find({ name: 'Paul' });
  console.log(result);
}

getUsers();
```

Output:
```
[
  {
    _id: new ObjectId("61fc5673d0ae011b8ebeae2e"),
    name: 'Paul',
    age: 33,
    hobbies: [ 'reading', 'writing' ],
    __v: 0
  }
]
```

#### Some useful methods
We can chain these methods to `User.find()`

Examples
* User.find().limit(5)
* User.find().limit(5).sort()

* .limit(5) to limit the number of results
* .sort({ name: -1 }) to sort by name in descendent order (-1 = descendent, 1 ascendent)
* select({ name: 1, age: 1 }) to select the specific properties to return