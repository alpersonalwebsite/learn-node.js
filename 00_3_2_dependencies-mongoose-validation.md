# Validation

More info: https://mongoosejs.com/docs/validation.html

We are going to take the example of the [previous chapter](./00_3_1_dependencies-mongoose.md)

```js
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  age: Number,
  hobbies: [String]
})
```

Now if you try to save a user without a name you will receive an error:

```js
async function createUser() {
  const user = new User({
    age: 34,
    hobbies: ['reading', 'writing']
  });

  try {
    const result = await user.save();
    console.log(result);
  } catch (err) {
      console.log(err);
  }
}

createUser();
```

Output:
```
Error: User validation failed: name: Path `name` is required.
```

We can also use the `document.validate(callback)` method:

```js
  try {
    user.validate(err => {
      if (err) {
        console.log(err);
      }
    })
  } catch (err) {
      console.log(err);
  }
}
```

## Built-in Validators

* All `SchemaTypes` have the built-in required validator. The required validator uses the SchemaType's checkRequired() function to determine if the value satisfies the required validator.
* `Numbers` have `min` and `max` validators.
* `Strings` have `enum`, `match`, `minLength`, and `maxLength` validators.

We can set required to:
1. A boolean
2. A function that returns a boolean

```js
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  age: Number,
  hobbies: [String],
  signature: {
    type: Boolean,
    required: function() {
      return this.age > 10;
    }
  }
});

async function createUser() {
  const user = new User({
    name: 'Peter',
    hobbies: ['reading', 'writing'],
    age: 20
  });

  try {
    const result = await user.save();
    console.log(result);
  } catch (err) {
      console.log(err);
  }
}

createUser();
```

In this example, if we try to add a user (object) with age greater tan 10 a signature is required.

```
Error: User validation failed: signature: Path `signature` is required.
```

## Custom Validators

```js
const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  hobbies: {
    type: Array,
    validate: {
      validator: function(value) {
        return value && value.length > 0;
      },
      message: 'User needs at least one hobby'
    }
  }
})
```

In this example, if we try to add a user without at least one hobby we will receive the following error:

```
Error: User validation failed: hobbies: User needs at least one hobby
```

## Async Custom Validators
For when we are dealing with async operations, like fetching data from an API, reading from a DB or disk...

