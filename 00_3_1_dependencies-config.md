# Config
(configuring applications)

---

* [Custom Environment Variables](#custom-environment-variables)

---

Install config: `npm install config`

Create a folder to hold the configuration files, example, config.
Then, create a file for each stage and add your configuration.

**config/development.json**

```json
{
  "name": "My Dev App"
}
```

**config/production.json**

```json
{
  "name": "My Prod App"
}
```

Then, in you application file, add:

```js
const config = require('config');

// ... more code
console.log(config.get('name'));
```

This will log to the console the name of your application.

## Custom Environment Variables

To enable custom environment variables, create a configuration file, **config/custom-environment-variables.json**
Then, we can map properties to environment variables.

```json
{
  "department": {
    "team": "TEAM"
  }
}
```

Then,
```
export TEAM=engineering
```

Then, in you application file, add:

```js
console.log(`Team: ${config.get('department.team')}`);
```

This will log to the console: `Team: engineering`