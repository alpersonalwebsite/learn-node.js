# Error Handling
More info: https://expressjs.com/en/guide/error-handling.html

Every time we are working with `promises` we need to handle the case of failure:

**Example 1: .then(data).catch(err)**

```js
const promise = new Promise(function(resolve, reject) {
  // async operations
  setTimeout(() => {
    //resolve({ data: 'This is the data' });
    reject(new Error('Something went wrong!'));
  }, 1000);
})

promise
  .then(result => console.log(result))
  .catch(err => console.log(err));

```

**Example 2: try/catch(err)**
```js
const promise = new Promise(function(resolve, reject) {
  // async operations
  setTimeout(() => {
    //resolve({ data: 'This is the data' });
    reject(new Error('Something went wrong!'));
  }, 1000);
})

async function getData() {
  try {
    console.log(await promise);
  } catch (err) {
    console.log(err)
  }
}

getData();
```

**How to handle unhandled exceptions in Node?**
This occurs when we have an exception that it is not handled by .catch()

```js
process.on('uncaughtException', (err, origin) => {
  console.log('Uncaught Exception', err);
  process.exit(1);
});

throw new Error('Something went wrong!');
```

Output:
```
Uncaught Exception Error: Something went wrong!
    at Object.<anonymous> (/Users/your-username/project/index.js:29:7)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
    at node:internal/main/run_main_module:17:47
```

Since we are catching the exception the process will not terminate. However, we should always terminate the process after `uncaughtException` and that's why we use `process.exit(1);`

**How to handle promises rejections that we are not catching?**
This will handle promise rejections that are not handled.

```js
process.on('unhandledRejection', (err, origin) => {
  console.log('Unhandled Rejection', err);
  process.exit(1);
});

Promise.reject(new Error('Failed'))
```

Output:
```
Unhandled Rejection Error: Failed
    at Object.<anonymous> (/Users/your-username/project/index.js:35:16)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
    at node:internal/main/run_main_module:17:47
```

Since we are catching the exception the process will not terminate. However, we should always terminate the process after `unhandledRejection` and that's why we use `process.exit(1);`

**IMPORTANT:** For both events, `unhandledRejection` and `uncaughtException` WE WANT to exit the process.

> 'uncaughtException' is a crude mechanism for exception handling intended to be used only as a last resort. The event should not be used as an equivalent to On Error Resume Next. Unhandled exceptions inherently mean that an application is in an undefined state. Attempting to resume application code without properly recovering from the exception can cause additional unforeseen and unpredictable issues. [NodeJS docs](https://nodejs.org/api/process.html#warning-using-uncaughtexception-correctly)

## Handle rejected promises by route

Example:
```js
// Get ALL users
router.get('/', async function (req, res) {
  try {
    const users = await User.find();
    res.send(users);
  } catch (err) {
    console.log(err);
    res.status(500).send();
  }
});
```

## Error handler middleware

We create the middleware:

src/middleware/error.js

```js
function error(err, req, res, next) {
  res.status(500).send();
}

module.exports = error;
```

In src/index.js we register the middleware as the last middleware function.

```js
const error = require('./middleware/error');


app.use('/', indexRoute);
app.use('/users', usersRoutes);
app.use('/products', productsRoutes);
app.use('/auth', authRoutes);

app.use(error);
```

We create a new middleware to handle promises (and its rejections)

src/middleware/handlePromisesRejections.js

```js
function handlePromisesRejections(routeHandler) {
  // we are returning a new route handler
  return async (req, res, next) => {
    try {
      await routeHandler(req, res);
    } catch (err) {
        next(err);
    }
  }
}

module.exports = handlePromisesRejections;
```

Then, in our routes:
(we don't need to add to each route try/catch())

```js
const handlePromisesRejections = require('../middleware/handlePromisesRejections');



router.get('/', handlePromisesRejections(async function (req, res) {
  const users = await User.find();
  res.send(users);
}));
```

We pass to `handlePromisesRejections()` middleware the route handler `async function (req, res) { ... }`
handlePromisesRejections() will return a new route handler where the our route handler reference (the one we passed) will be called.

## Use NPM package like ExpressJS Async Errors
More info: https://www.npmjs.com/package/express-async-errors

This is a very minimalistic and unintrusive hack. Instead of patching all methods on an express Router, it wraps the Layer#handle property in one place, leaving all the rest of the express guts intact.

The idea is that you require the patch once and then use the 'express' lib the usual way in the rest of your application.

We load the module in our main file (example, index.js)

**IMPORTANT:** When importing, if you are also importing `routes handlers` be sure `express-async-errors` is at the top.

Example:

```js
const morgan = require('morgan');
const winston = require('winston');

require('express-async-errors');

const indexRoute = require('./routes/index');
const usersRoutes = require('./routes/users');
const productsRoutes = require('./routes/products');
const authRoutes = require('./routes/auth');

const express = require('express');
const app = express();
```