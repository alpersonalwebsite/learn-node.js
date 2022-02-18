# Winston
For logging purposes.

We can log messages (aka, transport) to:
* Console
* HTTP (make an HTTP request to an endpoint)
* File

Also, to the following services with the attached plugins:
* MongoDB: https://www.npmjs.com/package/winston-mongodb
* Redis: https://www.npmjs.com/package/winston-redis

Install winston: `npm install winston`

**Logging levels** in winston conform to the severity ordering specified by RFC5424: severity of all levels is assumed to be numerically ascending from most important to least important.

```js
const levels = {
  error: 0,
  warn: 1,
  info: 2,
  http: 3,
  verbose: 4,
  debug: 5,
  silly: 6
};
```

Example: 

src/middleware/error.js

```js
const { createLogger, format, transports } = require('winston');

const logger = createLogger({
transports:
    new transports.File({
    filename: 'logs.log',
    format:format.combine(
        format.timestamp({format: 'MMM-DD-YYYY HH:mm:ss'}),
        format.align(),
        format.printf(info => `${info.level}: ${[info.timestamp]}: ${info.message}`),
        format.metadata()
    )}),
});

function error(error, req, res, next) {
  res.status(500).send('Internal Server Error');
  logger.error(`${error.status || 500} - ${res.statusMessage} - ${error.message} - ${req.originalUrl} - ${req.method} - ${req.ip}`, { metadata: error });
}

module.exports = error500;
```

src/index.js 

```js
const error = require('./middleware/error');


// It must be the last middleware
app.use(error);

app.listen(port, function() {
  console.log(`Running in localhost:${port}`);
});
```

Example output:

logs.log
```
error: Feb-17-2022 10:01:27: 	500 - Internal Server Error - BROKEN - /products - GET - ::ffff:127.0.0.1
```

## Error examples

### Logging 500 errors

In the previous example we are producing a 500 error throwing an error in one of our route handlers:

src/routes/products.js

```js
router.get('/', async function (req, res) {
  throw new Error('BROKEN')
});
```

### Logging 404 errors

We can refactor our error middleware and handle both, `500` and `404`

src/middleware/error.js

```js
const { createLogger, format, transports } = require('winston');

const logger = createLogger({
transports:
    new transports.File({
    filename: 'logs.log',
    format:format.combine(
        format.timestamp({format: 'MMM-DD-YYYY HH:mm:ss'}),
        format.align(),
        format.printf(info => `${info.level}: ${[info.timestamp]}: ${info.message}`),
        format.metadata()
    )}),
});

const messageTemplate = (errorStatus, error, req, res) => {
  if (error) return `${errorStatus} - ${res.statusMessage} - ${error.message} - ${req.originalUrl} - ${req.method} - ${req.ip}`
  return `${errorStatus} - ${res.statusMessage} - ${req.originalUrl} - ${req.method} - ${req.ip}`
};

function error500(error, req, res, next) {
  res.status(500).send('Internal Server Error');
  logger.error(messageTemplate(error.status || 500, error, req, res), { metadata: error });
}

function error404(req, res, next) {
  res.status(404).send('Not Found');
  logger.error(messageTemplate(404, null, req, res));
}

module.exports.error500 = error500;
module.exports.error404 = error404;
```

src/index.js 

```js
const { error500, error404 } = require('./middleware/error');


// They must be the last middlewares
app.use(error500);
app.use(error404);

app.listen(port, function() {
  console.log(`Running in localhost:${port}`);
});
```

Sample output:

```
error: Feb-17-2022 10:53:42: 	404 - Not Found - /product - GET - ::ffff:127.0.0.1
error: Feb-17-2022 10:53:46: 	500 - Internal Server Error - BROKEN - /products - GET - ::ffff:127.0.0.1
```

## Winston MongoDB
A MongoDB transport for winston.

Install winston-mongodb: `npm install winston-mongodb`

Then we require `winston-mongodb`, set transport value as an array of transports and add the new transports.MongoDB({})

```js
const {
  createLogger,
  format,
  transports
} = require('winston');

require('winston-mongodb');

const logger = createLogger({
  transports: [
    new transports.File({
      filename: 'logs.log',
      format: format.combine(
        format.timestamp({
          format: 'MMM-DD-YYYY HH:mm:ss'
        }),
        format.align(),
        format.printf(info => `${info.level}: ${[info.timestamp]}: ${info.message}`),
        format.metadata()
      )
    }),
    new transports.MongoDB({
      level: 'error',
      db: 'mongodb://localhost:27017/logs',
      options: {
        useUnifiedTopology: true
      },
      collection: 'server_logs',
      format: format.combine(
        format.timestamp(),
        format.json(),
        format.metadata()
        ),
        
    })
  ]
});

const messageTemplate = (errorStatus, error, req, res) => {
  if (error) return `${errorStatus} - ${res.statusMessage} - ${error.message} - ${req.originalUrl} - ${req.method} - ${req.ip}`
  return `${errorStatus} - ${res.statusMessage} - ${req.originalUrl} - ${req.method} - ${req.ip}`
};

function error500(error, req, res, next) {
  res.status(500).send('Internal Server Error');
  logger.error(messageTemplate(error.status || 500, error, req, res), { metadata: error });
}

function error404(req, res, next) {
  res.status(404).send('Not Found');
  logger.error(messageTemplate(404, null, req, res));
}

module.exports.error500 = error500;
module.exports.error404 = error404;
```

Sample result:
A new database, `logs`, will be created. Also a new collection `server logs` containing the logs records.

```json
{
    "_id": {
        "$oid": "620ea232a2d4bd033ff90a6a"
    },
    "timestamp": {
        "$date": "2022-02-17T19:29:54.105Z"
    },
    "level": "error",
    "message": "500 - Internal Server Error - BROKEN - /products - GET - ::ffff:127.0.0.1",
    "meta": {
        "message": "BROKEN",
        "name": "Error",
        "stack": "Error: BROKEN\n    at /Users/your-username/project/routes/products.js:15:9\n    at newFn (/Users/your-username/project/node_modules/express-async-errors/index.js:16:20)\n    at Layer.handle [as handle_request] (/Users/your-username/project/node_modules/express/lib/router/layer.js:95:5)\n    at next (/Users/your-username/project/node_modules/express/lib/router/route.js:137:13)\n    at Route.dispatch (/Users/your-username/project/node_modules/express/lib/router/route.js:112:3)\n    at newFn (/Users/your-username/project/node_modules/express-async-errors/index.js:16:20)\n    at Layer.handle [as handle_request] (/Users/your-username/project/node_modules/express/lib/router/layer.js:95:5)\n    at /Users/your-username/project/node_modules/express/lib/router/index.js:281:22\n    at Function.process_params (/Users/your-username/project/node_modules/express/lib/router/index.js:341:12)\n    at next (/Users/your-username/project/node_modules/express/lib/router/index.js:275:10)"
    }
}
```

## Handling `uncaughtException` and `unhandledRejection`
You can find several examples in the docs: https://www.npmjs.com/package/winston
We can log and handle `uncaughtException` and `unhandledRejection`.

One way to do it:

### uncaughtException

```js
const logger = createLogger({
  transports: [
    //...
  ],
  exceptionHandlers: [
    new transports.File({ filename: 'exceptions.log' })
  ]
});
```

So if we throw an error in Node:

```js
const express = require('express');
const app = express();

throw new Error('Something went wrong!')
```

Sample output in `exceptions.log`

```log
{"date":"Thu Feb 17 2022 15:22:26 GMT-0800 (Pacific Standard Time)","error":{},"exception":true,"level":"error","message":"uncaughtException: Something went wrong!\nError: Something went wrong!\n    at Object.<anonymous> (/Users/your-username/project/index.js:36:7)\n    at Module._compile (node:internal/modules/cjs/loader:1101:14)\n    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)\n    at Module.load (node:internal/modules/cjs/loader:981:32)\n    at Function.Module._load (node:internal/modules/cjs/loader:822:12)\n    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)\n    at node:internal/main/run_main_module:17:47","os":{"loadavg":[3.33837890625,4.2021484375,4.654296875],"uptime":2362527},"process":{"argv":["/Users/aldiaz/.nvm/versions/node/v16.13.0/bin/node","/Users/your-username/project/index.js"],"cwd":"/Users/your-username/project","execPath":"/Users/aldiaz/.nvm/versions/node/v16.13.0/bin/node","gid":20,"memoryUsage":{"arrayBuffers":36428814,"external":43497520,"heapTotal":57565184,"heapUsed":32281584,"rss":73146368},"pid":78637,"uid":501,"version":"v16.13.0"},"stack":"Error: Something went wrong!\n    at Object.<anonymous> (/Users/your-username/project/index.js:36:7)\n    at Module._compile (node:internal/modules/cjs/loader:1101:14)\n    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)\n    at Module.load (node:internal/modules/cjs/loader:981:32)\n    at Function.Module._load (node:internal/modules/cjs/loader:822:12)\n    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)\n    at node:internal/main/run_main_module:17:47","trace":[{"column":7,"file":"/Users/your-username/project/index.js","function":null,"line":36,"method":null,"native":false},{"column":14,"file":"node:internal/modules/cjs/loader","function":"Module._compile","line":1101,"method":"_compile","native":false},{"column":10,"file":"node:internal/modules/cjs/loader","function":"Module._extensions..js","line":1153,"method":".js","native":false},{"column":32,"file":"node:internal/modules/cjs/loader","function":"Module.load","line":981,"method":"load","native":false},{"column":12,"file":"node:internal/modules/cjs/loader","function":"Module._load","line":822,"method":"_load","native":false},{"column":12,"file":"node:internal/modules/run_main","function":"Function.executeUserEntryPoint [as runMain]","line":81,"method":"executeUserEntryPoint [as runMain]","native":false},{"column":47,"file":"node:internal/main/run_main_module","function":null,"line":17,"method":null,"native":false}]}

```

### unhandledRejection

```js
const logger = createLogger({
  transports: [
    //...
  ],
  rejectionHandlers: [
    new transports.File({ filename: 'rejections.log' })
  ]
});
```

So if we throw an error in Node:

```js
const express = require('express');
const app = express();

throw new Error('Something went wrong!')
```

Sample output in `exceptions.log`