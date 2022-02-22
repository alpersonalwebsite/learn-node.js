# Debug

Install debug: `npm install debug`

```js
const appDebugger = require('debug')('app:general');
// const appDbDebugger = require('debug')('app:db');

if (app.get('env') === 'development') {
  app.use(morgan('tiny'));
  appDebugger('Debugging App');
}

// For DB related code
// appDbDebugger('Debugging App DB');
```

Then, we set the env variables:

```shell
export DEBUG=app:general,app:db
# to enable one export DEBUG=app:general
# to enable some export DEBUG=app:general,app:db
# to enable use export DEBUG=*
```

Output:

```
  app:general Debugging App +0ms
  app:db Debugging App DB +0ms
```