# Authentication and Authorization

<!-- 
TODO: explain each one
-->

Authentication -> WHO?
Authorization -> WHAT?

Our sample API (the one that we used in the previous lessons) has the following endpoints (and methods):

* GET /products
* POST /products
* GET /users
* POST /users

What happens if we just want certain people to have the capabilities to add and/or remove products?
For example, only authenticated users can add products and only users with enough permissions (authorization) can delete products.

## Authentication

### Sample route for creating a USER
http://127.0.0.1:9000/users

In this example, after creating the user, we are sending the JWT in the response header.
(There are no intermediate steps like validation)

We also have the endpoint to get the user information: http://127.0.0.1:9000/users/me

```js
const mongoose = require('mongoose');
const express = require('express');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const config = require('config');
const auth = require('../middleware/auth');

const { User, validateUser } = require('../models/user');

const router = express.Router();

// Get the own user information
router.get('/me', auth, async function (req, res) {
  const user = await User.findById(req.user._id).select('-password -__v');
  res.send(user);
});

// Create a user
router.post('/', async function (req, res) {

  const { error } = validateUser(req.body);
  if (error) return res.status(400).send(error.details[0].message);

  // check first if we have a user with that email
  let user = await User.findOne({ email: req.body.email });

  if (user) return res.status(400).send(`We have that user`);

  // salt and hash the user password
  const salt = await bcrypt.genSalt(10);
  const hash = await bcrypt.hash(req.body.password, salt);

  user = new User({
    name: req.body.name,
    email: req.body.email,
    password: hash
  });

  try {
    await user.save();
  } catch (err) {
    for (let e in err.errors) {
      console.log(err.errors[e].message)
    }
  }

  const token = jwt.sign({ _id: user.id, user: user.name }, config.get('jwt-private-key'));

  // we avoid returning the password  
  res.header('X-Auth-Token', token).send({
    _id: user._id,
    name: user.name,
    email: user.email
  });
});

module.exports = router;
```

Sample output:

Header:

```
X-Auth-Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjBiZDdlNDI4N2Y2NjliM2ViNDNjNWQiLCJ1c2VyIjoidXUzIiwiaWF0IjoxNjQ0OTQzMzMyfQ.Q8Kmtc6oSkwn7olTcVSkWTZV_Mzn59eRK8NCtEWcO0M
```

Body:
```json
{
  "_id":"620bd7e4287f669b3eb43c5d",
  "name":"uu3",
  "email":"3@u.com"
}
```

### Sample route for authenticating a USER
http://127.0.0.1:9000/auth

First, we need to set the secret or JWT private Key as an environment variable:

```shell
export JWT_PRIVATE_KEY=privatekey19M
```


```js
const mongoose = require('mongoose');
const express = require('express');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

const { User, validateUserAuth } = require('../models/user');

const router = express.Router();

// Create a user
router.post('/', async function (req, res) {

  const { error } = validateUserAuth(req.body);
  if (error) return res.status(400).send(error.details[0].message);

  let user = await User.findOne({ email: req.body.email });
  if (!user) return res.status(400).send({ data: `Invalid email or password` });

  const isValidPassword = await bcrypt.compare(req.body.password, user.password);
  if (!isValidPassword) return res.status(400).send({ data: `Invalid email or password` });

  const token = jwt.sign({ _id: user.id, user: user.name }, process.env.JWT_PRIVATE_KEY)

  res.send({ data: token });
});

module.exports = router;
```

The output will either be:

* 200 with the payload { "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MjBhZDUyZmYwYjZmMWZkNGVlNTI3MmYiLCJ1c2VyIjoidXUyIiwiaWF0IjoxNjQ0OTQxNzUwfQ.5UEI4aMoksWb7BafKIkhTMNtoWZw9C6lItjk-Vgs_8k" }

* 400 with the payload { "data": "Invalid email or password" }

### Sample way to protect a route with a middleware function

middleware/auth.js
```js
const jwt = require('jsonwebtoken');
const config = require('config');

function auth(req, res, next) {
  const token = req.header('X-Auth-Token');
  if (!token) return res.status(401).send('Unauthorized');

  try {
    // Another option: const decodedToken = jwt.verify(token, process.env.JWT_PRIVATE_KEY);
    const decodedToken = jwt.verify(token, config.get('jwt-private-key'));
    req.user = decodedToken;
    next();
    
  } catch (err) {
    res.status(400).send('Invalid token');
  }
}

module.exports = auth;
```

http://127.0.0.1:9000/products

```js
const mongoose = require('mongoose');
const express = require('express');
const auth = require('../middleware/auth');

const { Product, validateProduct } = require('../models/product');

const router = express.Router();

// Create a product
// We pass auth to be executed before the route handler
router.post('/', auth, async function (req, res) {

  const { error } = validateProduct(req.body);
  if (error) return res.status(400).send(result.error.details[0].message);

  const product = new Product({
    name: req.body.name,
    price: req.body.price
  });

  try {
    await product.save();
  } catch (err) {
    for (let e in err.errors) {
      console.log(err.errors[e].message)
    }
  }

  res.send(product);
});

module.exports = router;
```

Sample results...

1. If we don't pass a token: 401 -> `Unauthorized`
2. If we pass a wrong token: 400 -> `Invalid token`
3. If we pass a valid token we should be able to create a product 200 -> product

## Authorization

This is a basic example where we just have `registered user` and `admin`.

<!--
TODO: more complex example Auth0 ???
-->

We want only admins to be able to remove products.
DELETE -> http://127.0.0.1:9000/products/62058ce4665d8258848cd7bd

Our userSchema

```js
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    minlength: 3,
    maxlength: 20
  },
  email: {
    type: String,
    unique: true
  },
  password: {
    type: String,
    required: true
  },
  isAdmin: Boolean
});

userSchema.methods.generateAuthToken = function() {
  return jwt.sign({ _id: this.id, user: this.name, isAdmin: this.isAdmin }, config.get('jwt-private-key'));
}
```

Our admin middleware:

```js
function admin(req, res, next) {
  // req.user is coming from auth middleware which runs first
  if (!req.user.isAdmin) return res.status(403).send('Access Denied');

  next();
}

module.exports = admin;
```

Our route handler:

```js
// Delete a product
router.delete('/:id', [ auth, admin ], async function (req, res) {

  let product;
  try {
    product = await Product.findByIdAndRemove({
      _id: req.params.id
    });
  } catch (err) {
    console.log(err);
  }

  if (!product) return res.status(404).send(`We don't have that product`);

  res.send(product);
});
```

Sample results...

1. If we don't pass a token: 401 -> `Unauthorized`
2. If we pass a wrong token: 400 -> `Invalid token`
3. If the token is valid but the user is not an admin: 403 -> `Access Denied`
4. If we pass a valid token we should be able to delete a product 200 -> product
