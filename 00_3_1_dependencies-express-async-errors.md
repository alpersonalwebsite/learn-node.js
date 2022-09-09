# ExpressJS Async Errors
More info: https://www.npmjs.com/package/express-async-errors

This is a very minimalistic and unintrusive hack. Instead of patching all methods on an express Router, it wraps the Layer#handle property in one place, leaving all the rest of the express guts intact.

The idea is that you require the patch once and then use the 'express' lib the usual way in the rest of your application.

Install ExpressJS Async Errors: `npm install express-async-errors`

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