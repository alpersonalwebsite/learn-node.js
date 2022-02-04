# MongoDB
MongoDB is a document database with the scalability and flexibility that you want with the querying and indexing that you need.

First, you need to have `MongoDB` installed.
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/


## Installing MongoDB 5.0 Community Edition in MAC

```shell
brew tap mongodb/brew

brew install mongodb-community@5.0
```

Now, start the `mongo daemon`

```
mongod
```

## Installing MongoDB Client

Download and install a MongoDB client. For example: https://www.mongodb.com/products/compass

Then, connect using the default connection string.
```
mongodb://localhost:27017/?readPreference=primary&appname=MongoDB%20Compass&directConnection=true&ssl=false
```

For the following steps, you will need to have a database and collection created.
Using `Compass`, create a new database (`my-app`) and a new collection (`users`).

## Populating data

In this example we are going to populate fake data generated using `mockaroo`: https://www.mockaroo.com/

```shell
mongoimport --db my-app1 --collection users --file 00_4_1_mock-data.json --jsonArray
```

Output: 
```
2022-02-04T09:05:13.270-0800	connected to: mongodb://localhost/
2022-02-04T09:05:13.286-0800	100 document(s) imported successfully. 0 document(s) failed to import.
```