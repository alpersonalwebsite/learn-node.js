# Node Module System

---

* [Creating a module](#creating-a-module)
* [Node built-in modules](#node-built-in-modules)
  + [Path module](#path-module)
  + [OS module](#os-module)
  + [File System module](#file-system-module)
  + [Events module](#events-module)
  + [HTTP module](#http-module)
* [NPM](#npm)
  + [SemVer (Semantic versioning)](#semver--semantic-versioning-)
  + [Publishing a package in NPM Registry](#publishing-a-package-in-npm-registry)

---

Every file in a Node application is considered a module.
The variables and functions defined in a file/module are scoped to that file/module. 
If we want to use a variable or function outside a module we need to export it and make it public

## Creating a module

Main module: `app.js`

```js
const greet = require('./greet');

greet.sayHi(`Hola`);
```

`greet.js`

```js
function sayHi(message) {
  console.log(message);
}

module.exports.sayHi = sayHi;
```

```
node app.js
```

Note: If we are exporting just one method, we can do: `module.exports = sayHi;`

If we do that, greet will reference the function, so we will use `greet(msg)` instead of `greet.sayHi(msg)`

---

Node wraps the code of our modules with a function wrapper that has the following structure:

```js
(function (exports, require, module, __filename, __dirname) {
  // our code
});
```

## Node built-in modules

```js
const module = require(module);
```

### Path module

### OS module

### File System module

```js
const fs = require('fs');

// Sync
const files = fs.readdirSync('./');
console.log(files);

// Async
fs.readdir('./', function(err, files) {
  if (err) console.log(err);
  else console.log(files);
})
```

### Events module

An event is a signal that something has happened.

```js
const EventEmitter = require('events');

// Create a new instance of the class EvenEmitter

const emitter = new EventEmitter();

// Registering a listener for newUser
emitter.on('newUser', function(e) {
  console.log(`newUser event was emitted: ${JSON.stringify(e)}`);
})

// Raise event where newUser is the name of the event 
emitter.emit('newUser', { name: 'Peter', username: 'PP' });
```

Output:
```
newUser event was emitted: {"name":"Peter","username":"PP"}
```

**Note**: remember to `register first the listener`, then emit the event.

The previous example works well for one-file project. However, when we work with several files (or modules), things change a bit.

We need to use the `same object` for registering an event listener and for emitting the event.
So, we are not to create an emitter on each module (like `const emitter = new EventEmitter();`) because we would end with 2 different objects.

We do the following:
1. Create a class that extends the EventEmitter class (this new class will contain all the functionality of EventEmitter plus our methods)
2. Inside the class we use this (`this.emit()`) which refers to the own class.
3. In the other file or module we use an instance of our class instead of an instance of EventEmitter.

Main module a.js

```js
const EventEmitter = require('events');

const Greet = require('./greet');
const greet = new Greet();

// we register the listener on greet object
greet.on('greetSent', function(e) {
  console.log(`greetSent event was emitted: ${JSON.stringify(e)}`);
})

greet.sayHi(`Hola`);
```

Module b.js

```js
const EventEmitter = require('events');

class Greet extends EventEmitter {
  sayHi(message) {
    console.log(message);

    this.emit('greetSent', { date: new Date() })
  }

}

module.exports = Greet;
```

### HTTP module

```js
const http = require('http');

const server = http.createServer(function(req, res) {
  if (req.url === '/') {
    res.write(`Home`)
    res.end();
  }
});

server.listen(9000);

console.log(`Running in localhost:9000`);
```

If you go to: http://localhost:9000/ you will see `Home`

---

## NPM

Initializing a project accepting defaults `npm init -y`

When we require a package, Node first looks in...

1. Code modules
2. Local modules (our files or modules)
3. node_modules

Note: if you are using source control like `git`, remember to exclude `node_modules/` in your `.gitignore` file.

If you want to check for outdated dependencies: `npm outdated`

We have 2 types of dependencies: 
1. `dependencies` (our app dependencies)
2. `devDependencies` (dependencies that we don't want in our production bundle)

### SemVer (Semantic versioning)

**@Major.Minor.Patch**

Example: express@4.17.2

* Patch for bugs fixes @4.17.3
* Minor for non breaking changes @4.18.0
* Major for breaking changes @5.0.0

If yoy use `4.17.2` that particular version will installed.
If you use `~` (~4.17.2) new patch releases will be installed.
If you use `^` (^4.17.2) new patch and minor releases will be installed.

### Publishing a package in NPM Registry

After creating an account (npm add-user) log into the registry: `npm login`

Then, `npm publish`
If you are updating a previously published package, depending on the change, you will update the SemVer (npm version major, as an example) and execute `npm publish` again.

After this, you can install your package with `npm install your-package-name`

