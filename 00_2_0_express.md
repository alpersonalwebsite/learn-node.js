# Express

Install Express: `npm install express`

---

* [Basic Express App](#basic-express-app)
* [Accessing environment variable](#accessing-environment-variable)
* [Using route parameters](#using-route-parameters)
* [Using query string parameters](#using-query-string-parameters)
* [CRUD operations](#crud-operations)
  + [Create or post()](#create-or-post)
  + [Read or .get()](#read-or-get)
  + [Update or .put()](#update-or-put)
  + [Delete or .delete()](#delete-or-delete)
* [Middleware](#middleware)
  + [Creating a middleware](#creating-a-middleware)
* [Sample Express App Architecture](#sample-express-app-architecture)

---

## Basic Express App

```js
const express = require('express');
const app = express();

const port = process.env.PORT || 9000;

app.get('/', function(req, res) {
  res.send(`Home`);
});

app.listen(port, function() {
  console.log(`Running in localhost:${port}`);
});
```

If you go to: http://localhost:9000/ you will see `Home`

## Accessing environment variable

We can access to the environment variables through the object `process.env`
Example: `process.env.NODE_ENV`. This will return undefined or the value of NODE_ENV

In Express we can use `app.get('env')` to access to the `env`. If there's no `NODE_ENV` defined, it will return `development`, else, whatever `NODE_ENV` holds.

In MAC you can...
1. Set an env variable with: export NODE_ENV=production
2. Unset an env variable with: unset NODE_ENV

You can set an environment variable and execute Node at the same time:

```shell
NODE_ENV=production node index.js
```

## Using route parameters

```js
app.get('/users/:id', function(req, res) {
  res.send(`The route parameter is ${req.params.id}`);
});
```

If you go to: http://localhost:9000/users/6 you will see `The route parameter is 6`

We can define routes with route parameters:

```js
app.get('/users/:userId/hobbies/:hobbyId', function(req, res) {
  res.send(req.params);
});
```

If you go to http://localhost:9000/users/6/hobbies/2

```json
{
"userId": "6",
"hobbyId": "2"
}
```

## Using query string parameters

```js
app.get('/users', function(req, res) {
  res.send(req.query);
});
```

If you go to http://localhost:9000/users?sort=desc&status=active

```json
{
"sort": "desc",
"status": "active"
}
```

## CRUD operations

* C > Create > Post
* R > Read > Get
* U > Update > Put
* D > Delete > Delete

### Create or post
.post(/collection)

This will add a new resource (user) to the collection (users) 

First enable parsing of JSON `app.use(express.json());`
From Express docs: "This is a built-in middleware function in Express. It parses incoming requests with JSON payloads and is based on body-parser".

```js
app.post('/users', function(req, res) {

  if (!req.body.name) return res.status(400).send(`Please provide a name`);

  const user = {
    id: users.length + 1,
    name: req.body.name
  }

  users.push(user);

  res.send(user);
});
```

### Read or .get
.get(/collection) or .get(collection/resource)

This will return an array of objects (or users)

```js
app.get('/users', function(req, res) {
  res.send(users);
});
```

or...

This will return an object (user) or 404 error if there's no user with that id.

```js
app.get('/users/:id', function(req, res) {
  const user = users.find(user => user.id == parseInt(req.params.id));
  if (!user) return res.status(404).send(`We don't have that user`);

  res.send(user);
});
```

### Update or .put
.put(/collection/resource)

This will update the resource (user) of the collection (users).
It will return 404 if there is no user with that id, 400 if the payload doesn't contain a name property and the updated user if the operation was successful.

```js
app.put('/users/:id', function(req, res) {
  let user = users.find(user => user.id == parseInt(req.params.id));
  if (!user) return res.status(404).send(`We don't have that user`);

  if (!req.body.name) return res.status(400).send(`Please provide a name`);

  user.name = req.body.name;

  res.send(user);
});

```

### Delete or .delete
.delete(/collection/resource)

This will delete a resource from the collection.
It will return 404 if there is no user with that id and the deleted user if the operation was successful.

```js
app.delete('/users/:id', function(req, res) {
  let user = users.find(user => user.id == parseInt(req.params.id));
  if (!user) return res.status(404).send(`We don't have that user`);
  
  const userIndex = users.indexOf(user);
  users.splice(userIndex, 1);

  res.send(user);
});
```

## Middleware

From Express docs: "Express is a routing and middleware web framework that has minimal functionality of its own: An Express application is essentially a series of middleware function calls".

A middleware is a function that takes a request object and..
1. Returns a response
2. Call the next middleware function in the stack

Example: route handlers.
The route handlers take a request and return a response.

```js
app.get('/users', function(req, res) {
  res.send(users);
});
```

**Built-in middleware in Express**

* `express.static` serves static assets such as HTML files, images, and so on. `app.use(express.static(public))`
* `express.json` parses incoming requests with JSON payloads. NOTE: Available with Express 4.16.0+ `app.use(express.json())`
* `express.urlencoded` parses incoming requests with URL-encoded payloads. NOTE: Available with Express 4.16.0+ `app.use(express.urlencoded({ extended: true }))`

Express middleware: https://expressjs.com/en/resources/middleware.html

### Creating a middleware

```js
// next refers to the next middleware
app.use(function(req, res, next) {
  console.log(`Log enabled!`)

  // we need to return a response or next or the request will end hanging
  next();
});
```

## Sample Express App Architecture 

<!-- 
TODO: Research more about this and try to find a cross project structure
-->

src/
  config/
  middleware/
  public/
  routes/
    index.js
    users.js
  views/
    index.pug
  index.js