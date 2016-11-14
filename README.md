# kvplus-js
Specs for a simple Key/Value (plus Secondary Indexes) store API, for Node.js/Javascript

### Contents

* [Creating a Store Instance](#creating-a-store-instance)
* **CRUD API**
  * [`exists()`](#exists)
  * [`get()`](#get)
  * [`put()`](#put)
  * [`del()`](#del)
* **Secondary Index API**
  * [`createIndex()`](#createindex)
  * [`findBy()`](#findby)

### Creating a Store Instance

Usage:

```js
var options = {
  path: './db'
}
var store = new KVPlusStore(options)
```

##### Constructor Options

* `path` - (Optional) For filesystem based stores, specifies the directory
  where the store will be created.

### CRUD API

#### exists()

`Promise<Boolean> exists (string collectionName, string key)`

Checks whether an object exists in the collection, for a given key.

Usage:

```js
store.exists('users', 'u1')
  .then(found => {
    if (found) {
      // user exists!
    } else {
      // not found
    }
  })
```

#### get()

`Promise<Object> get (string collectionName, string key)`

Retrieves the serialized object from the store, from the given collection.

Usage:

```js
store.get('users', 'u1')
  .then(retrievedUser => {
    console.log(retrievedUser)  // user object with key 'u1' retrieved
  })
```

#### put()

`Promise<Object> put (string collectionName, string key, Object data)`

Writes an object to the store, and returns the object that was written. (If an
object already exists for that key, in that collection, it gets overwritten.)

Usage:

```js
store.put('users', 'u2', { name: 'Alice' })
  .then(storedUser => {
    assert(storedUser.name === 'Alice')
  })
```

### del()

`Promise<Boolean> del (string collectionName, string key)`

Attempts to delete an object for a given collection and key. If the object
existed and the delete succeeded, resolves to `true`. If the object did not
exist, resolves to `false`.

Usage:

```js
store.del('users', 'u2')
  .then(() => {
    // user deleted. In this case, we don't care about whether the user actually
    // existed, so we're not going to check the boolean result of the operation
  })
```

## Secondary Index API

### createIndex()

`Promise<> createIndex (string collectionName, string property)`

Creates a secondary index on a given object property, and enables subsequent
secondary queries on that field. Should be idempotent (if the index already
exists, results in no-op).

Usage:

```js
store.createIndex('users', 'email')
  .then(() => {
    // secondary index created on the 'email' property of users objects.
    // you can now perform findBy() queries on this property
  })
```

### findBy()

`Promise<Array<Object>> findBy (string collectionName, string property, string value)`

Performs a secondary index query (exact matches only, for now) on a given
existing index and value.

Usage:

```js
store.findBy('users', 'email', 'alice@example.com')
  .then(results => {
    console.log(results)  // -> [ { name: 'Alice', email: 'alice@example.com'} ]
  })
```

#### Questions/Design Decisions

* [ ] Should there be a `list(collectionName)` api method? issue #1
* [ ] Should there be a `createCollection(name)` api method, or do we want to
  limit the backends/implementations to lazy-initializing collections in the
  course of other operations? issue #2
* [ ] Should there be a way to list all collections? (created explicitly or
  lazily) issue #3
