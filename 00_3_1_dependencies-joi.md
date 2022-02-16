# Joi
(for validation purposes)

Install Joi: `npm install joi`

In the following example we are validating that the request body includes a `user`. If not, we retrieve an error.

```js
// it returns the class Joi
const Joi = require('joi');

// ... more code here ...

app.post('/users', function(req, res) {

  // shape of user object
  const userSchema = Joi.object({
    name: Joi.string().required()
  });

  const result = userSchema.validate(req.body);

  if (result.error) {
    res.status(400).send(result.error.details[0].message);
    return;
  }

  const user = {
    id: users.length + 1,
    name: req.body.name
  }

  users.push(user);

  res.send(user);
});
```

## Joi ObjectId
If we need to validate a MongoDB ObjectId.

Install Joi: `npm install joi-objectid`

<!-- 
TODO:
-->

## Joi Password Complexity
Creates a Joi object that validates password complexity.

Install Joi: `npm install joi-password-complexity`

```js
function validateUser(user) {
  const userSchema = Joi.object({
    name: Joi.string().min(3).max(20).required(),
    email: Joi.string().required().email(),
    password: passwordComplexity({
      min: 8,
      max: 26,
      lowerCase: 1,
      upperCase: 1,
      numeric: 1,
      symbol: 1,
      requirementCount: 4
    })
  });

  return userSchema.validate(user);
}

module.exports.validateUser = validateUser;
```