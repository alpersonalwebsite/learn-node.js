# Mongoose Relationships

## Reference Based Relationships (Normalization)

* Pros: consistency (we update in one place)
* Cons: requires extra queries

```js
let store = {
  name: 'My store'
}

let product = {
  store: 'id'
}
```

We have 2 documents: `store` and `product` (each belonging to its own collection, stores and products).
The product object references the id of a store document in the stores collection.

This relationship is not enforced (like in relational databases). There's no real relationship between the documents.

## Embedded Documents Relationships (Denormalization)

* Pros: performance -> single query (store is inside product)
* Cons: consistency (we need to update in several places since we are going to have multiple products)

In this case we embed a store document inside the product document. 

```js
let product = {
  store: {
    name: 'My store'
  }
}
```

---

## Reference Based Relationships (Normalization) real example

First, create the following collections: `stores` and `products` in our `my-app` db.

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/my-app')
  .then(() => console.log('Connected to MongoDB!'))
  .catch(err => console.error(err));

// Schemas
const storeSchema = new mongoose.Schema({
  name: String
});

const productSchema = new mongoose.Schema({
  name: String,
  price: Number,
  store: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Store'
  }
});

// Models
const Store = mongoose.model('Store', storeSchema);
const Product = mongoose.model('Product', productSchema);


async function createStore({ name }) {
  console.log(name);
  const store = new Store({ name })

  try {
    const result = await store.save();
    console.log(result);
  } catch (err) {
      console.log(err);
  }
}

async function createProduct({ name, price, store }) {
  const product = new Product({ name, price, store })

  try {
    const result = await product.save();
    console.log(result);
  } catch (err) {
      console.log(err);
  }
}

async function listProducts() { 
  const products = await Product
    .find()
    .populate('store', 'name -_id')
    .select('name price store');
  console.log(products);
}



createStore({ name: 'Store 1' });

// We pass the id of the previously created store
createProduct({ name: 'Product 1', price: 10, store: '62042cceadfeb2715311639c' });

listProducts();
```

Output:

```
[
  {
    _id: new ObjectId("62042cf5623bf000a451c132"),
    name: 'Product 1',
    price: 10,
    store: { name: 'Store 1' }
  }
]
```

In the `product schema` we defined the property `store` which references to a store of the stores collection.
when we use the `.populate(property, propertiesToInCludeOrExclude)` method and pass the store field, mongoose will query the stores collection and pull the document (store) that has that store id. If there's no matching id, it will return `null` (example: `store: null`)
As the second argument of populate() we can pass the properties we want to include or exclude (-property).

Example: `.populate('store', 'name -_id')` will show the name of the store but not its id.

More information about `Populate`: https://mongoosejs.com/docs/populate.html


## Embedded Documents Relationships (Denormalization) real example

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/my-app')
  .then(() => console.log('Connected to MongoDB!'))
  .catch(err => console.error(err));

// Schemas
const storeSchema = new mongoose.Schema({
  name: String
});

const productSchema = new mongoose.Schema({
  name: String,
  price: Number,
  store: storeSchema
});

// Models
const Store = mongoose.model('Store', storeSchema);
const Product = mongoose.model('Product', productSchema);


async function createProduct({
  name,
  price,
  store
}) {
  const product = new Product({
    name,
    price,
    store
  })

  try {
    const result = await product.save();
    console.log(result);
  } catch (err) {
    console.log(err);
  }
}

createProduct({
  name: 'Product 1',
  price: 10,
  store: new Store({
    name: 'Store 1'
  })
});
```

Output:
```
{
  name: 'Product 1',
  price: 10,
  store: { name: 'Store 1', _id: new ObjectId("62058c140f27cb5976a5358d") },
  _id: new ObjectId("62058c140f27cb5976a5358e"),
  __v: 0
}
```