## Mongoose
Object Data Modeling (ODM) library for MongoDB and Node.js.

---

- [Mongoose](#mongoose)
- [Connecting to MongoDB](#connecting-to-mongodb)
- [Initial terminology](#initial-terminology)
- [Creating a Schema](#creating-a-schema)
- [SchemaType Options](#schematype-options)
  * [Some useful SchemaType Options](#some-useful-schematype-options)
- [Creating a Model](#creating-a-model)
- [Methods and Statics](#methods-and-statics)
  * [Instance method example](#instance-method-example)
  * [Static method example](#static-method-example)
- [CRUD operations](#crud-operations)
  * [Creating a document](#creating-a-document)
  * [Retrieving ALL documents in a collection](#retrieving-all-documents-in-a-collection)
  * [Retrieving some documents](#retrieving-some-documents)
    + [Some useful methods](#some-useful-methods)
  * [Updating a document](#updating-a-document)
    + [Query and update](#query-and-update)
    + [Update](#update)
  * [Deleting documents](#deleting-documents)
    + [Deleting one document](#deleting-one-document)
      - [Deleting retrieving the deleted document](#deleting-retrieving-the-deleted-document)
      - [Deleting without retrieving](#deleting-without-retrieving)
    + [Deleting several documents](#deleting-several-documents)
- [MongoDB Query operators](#mongodb-query-operators)
  * [Comparison Query Operators](#comparison-query-operators)
  * [Logical Query Operators](#logical-query-operators)
  * [Regular Expressions](#regular-expressions)

---

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
});
```

## SchemaType Options

You can declare a schema type using the type directly, or an object with a type property.

```js
const userSchema = new mongoose.Schema({
  name: String,
});

// or

const userSchema = new mongoose.Schema({
  name: {
    type: String
  }
});
```

### Some useful SchemaType Options

* For ALL types

  - default: default value
  - validate: function, adds a validator function for this property
  - get: function, defines a custom getter for this property using Object.defineProperty(). Called when we get the value of the property.
  - set: function, defines a custom setter for this property using Object.defineProperty(). Called when we set the value of the property.


* For STRING

  - lowercase: boolean, whether to always call .toLowerCase() on the value
  - uppercase: boolean, whether to always call .toUpperCase() on the value
  - trim: boolean, whether to always call .trim() on the value
  - match: RegExp, creates a validator that checks if the value matches the given regular expression
  - enum: Array, creates a validator that checks if the value is in the given array.

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

## Methods and Statics

Each Schema can define instance and static methods for its model.

Remember:
- `static methods` are methods that exists on our Model (or class)
- `instance methods` are methods that exists on the objects (instances) created by the Class

### Instance method example

Schema and model file
```js
const userSchema = new mongoose.Schema({
  name: String
});

userSchema.methods.generateAuthToken = function() {
  return jwt.sign({ _id: this.id, user: this.name, isAdmin: this.isAdmin }, config.get('jwt-private-key'));
}

const User = mongoose.model('User', userSchema);
```

File
```js
const user = new User({ name: 'Peter' });

const token = user.generateAuthToken();
```

### Static method example

Schema and model file
```js
const productSchema = new mongoose.Schema({
  name: String,
  price: Number,
});

productSchema.statics.delete = function(productId) {
  return this.findByIdAndRemove({
    _id: productId
  });
}

const Product = mongoose.model('Product', productSchema);
```

File
```js
const product = await Product.delete(req.params.id);
```

## CRUD operations

### Creating a document

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
* User.find().limit(...).sort(...).select(...)

* .limit(5) -> to limit the number of results
* .sort({ name: -1 }) -> to sort by name in descendent order (-1 = descendent, 1 ascendent)
* .select({ name: 1, age: 1 }) -> to select the specific properties to return
* .count() -> retrieves the number of documents that match our object
* skip(number).limit(number) -> we can use this for pagination

Example:

```js
const pageNumber = 4;
const pageSize = 15;

const result = await User
  .find({ name: 'Paul' })
  .skip((pageNumber -1) * pageSize)
  .limit(pageSize);
```

### Updating a document

#### Query and update
```js
async function updateUser(id) {
  const user = await User.findById(id);

  if (!user) return;

  user.set({
    age: 0
  })

  const result = await user.save();
  console.log(result);
}

updateUser('61fd5d9835329f05cd0ba119');
```

Output:

```
{
  name: 'Peter',
  age: 0,
  hobbies: [ 'reading', 'writing' ],
  _id: new ObjectId("61fd5d9835329f05cd0ba119"),
  __v: 0
}
```

#### Update
More info about Update Operators: https://docs.mongodb.com/manual/reference/operator/update/

Note: If you want to retrieve the updated document, you can use `findByIdAndUpdate()` with the new option instead of `.update()`

```js
async function updateUser(id) {
  const result = await User.update(
    {
      _id: id
    },
    {
      $set: { 
        age: 1
       }
    }
  );

  console.log(result);
}

updateUser('61fd5d9835329f05cd0ba119');
```

Output:

```
{
  acknowledged: true,
  modifiedCount: 1,
  upsertedId: null,
  upsertedCount: 0,
  matchedCount: 1
}
```

Example: `findByIdAndUpdate()`

```js

async function updateUser(id) {
  const result = await User.findByIdAndUpdate(
    id,
    {
      $set: { 
        age: 2
       }
    },
    {
      new: true
    }
  );

  console.log(result);
}

updateUser('61fd5d9835329f05cd0ba119');
```

Output:

```
{
  _id: new ObjectId("61fd5d9835329f05cd0ba119"),
  name: 'Peter',
  age: 2,
  hobbies: [ 'reading', 'writing' ],
  __v: 0
}
```

### Deleting documents

#### Deleting one document

##### Deleting retrieving the deleted document

```js
async function deleteUser(id) {
  const user = await User.findByIdAndRemove({ _id: id });
  console.log(user);
}

deleteUser('61fc5673d0ae011b8ebeae2e');
```

Output:
```
{
  _id: new ObjectId("61fc5673d0ae011b8ebeae2e"),
  name: 'Paul',
  age: 33,
  hobbies: [ 'reading', 'writing' ],
  __v: 0
}
```

##### Deleting without retrieving

```js
async function deleteUser(id) {
  const result = await User.deleteOne({ _id: id });
  console.log(result);
}

deleteUser('61fd56b9943778743557f0b0');
```

Output:
```
{ deletedCount: 1 }
```

#### Deleting several documents

```js
async function deleteUsers() {
  const result = await User.deleteMany({ name: 'Peter' });
  console.log(result);
}

deleteUsers();
```

Output:
```
{ deletedCount: 12 }
```

## MongoDB Query operators

### Comparison Query Operators
These operators are part of MongoDB and are they available in Mongoose since Mongoose is built on top of MongoDB driver.

More info: https://docs.mongodb.com/manual/reference/operator/query-comparison/

* $eq -> equal
* $gt -> greater than
* $gte -> greater than or equal
* $in 
* $lt -> less than
* $lte -> less than or equal
* $ne -> not equal
* $nin -> not in

Example: with age greater than 33

```js
async function getUsers() {
  const result = await User.find({ age: { $gt: 33 } });
  console.log(result);
}

getUsers();
```

We can use multiple operators. Example: with age greater than 33 and name equals to Wendy:

```js
const result = await User.find({ age: { $gt: 33, $eq: 'Wendy' } });
```

If we want to match several values we can use $in which matches any of the values specified in an array.

```js
const result = await User.find({ age: { $in: ['Wendy', 'Peter'] } });
```

### Logical Query Operators
More info: https://docs.mongodb.com/manual/reference/operator/query-logical/

* $and ->
* $not ->
* $nor ->
* $or ->

Example $or: this will retrieve users with name Wendy OR age equals to 34

```js
  const result = await User
    .find()
    .or([{ name: 'Wendy' }, { age: 34 }]);
```

### Regular Expressions
More info: https://docs.mongodb.com/manual/reference/operator/query/regex/

Example: all names starting with `p/P` (i is case insensitive)

```js
const result = await User.find({ name: /^P/i });
```

---

Full example:

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/my-app')
  .then(() => console.log('Connected to MongoDB!'))
  .catch(err => console.error(err));

const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  hobbies: [String]
})

// User is the singular name of the collection Users
const User = mongoose.model('User', userSchema);

async function createUser() {
  // With the User class we can create objects 
  const user = new User({
    name: 'Peter',
    age: 34,
    hobbies: ['reading', 'writing']
  });

  const result = await user.save();
  console.log(result);
}


async function getUsers() {
  const result = await User.find({ name: /^p/i });
  console.log(result);
}


createUser();
getUsers();
```