# kvplus-js
Specs for a simple Key/Value (plus Secondary Indexes) store API, for Node.js/Javascript

### Contents

* [Motivation](#motivation)
* [Store API](#store-api)
  * [Creating a Store Instance](#creating-a-store-instance)
  * [Initializing a Collection](#initializing-a-collection)
* [CRUD API](#crud-api)
  * [`exists()`](#exists)
  * [`get()`](#get)
  * [`put()`](#put)
  * [`del()`](#del)
* [Secondary Index API](#secondary-index-api)
  * [`createIndex()`](#createindex)
  * [`findBy()`](#findby)
* [Implementations](#implementations)

### Motivation

This K/V API was created to provide a pluggable persistence interface for the
[`oidc-op`](https://github.com/anvilresearch/oidc-op) OpenID Connect Identity
Provider library (to store user accounts, client registrations, access tokens,
and so on).

### Store API

#### Creating a Store Instance

Usage:

```js
var options = {
  path: './db',
  serialize: (obj) => { return JSON.stringify(obj) }
}
var store = new KVPlusStore(options)
```

##### Constructor Options

* `path` - (string) For filesystem based stores, specifies the directory
  where the store will be created. Optional.
* `serialize` - (Function) If storing objects other than strings, provide a
  serialization function that will turn an object into a string representation.
  Can also be provided for the whole store (in the store constructor), or
  overridden for each collection.

### Initializing a Collection

`Promise<> createCollection (string collectionName, Object options = {})`

Creates/initializes a collection in the store. This operation should be
*idempotent* -- if the collection already exists, it should result in a no-op.
Think of it as likely to run each time an app server starts up (instead of only
once, at install time or in a database migration).

##### Collection options

* `serialize` - (Function) If storing objects other than strings, provide a
  serialization function that will turn an object into a string representation.
  Can also be provided for the whole store (in the store constructor), or
  overridden for each collection.

Usage:

```js
let serialize = (obj) => { return JSON.stringify(obj) }
store.createCollection('users', { serialize })
  .then(() => {
    // collection created/initialized
  })
```

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

### Secondary Index API

#### createIndex()

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

#### findBy()

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

### Implementations

* [`kvplus-files`](https://github.com/solid/kvplus-files) Simple file-based
  JSON store.

The following are in-progress implementations.

* In-memory KVPlus store. (For reference / testing)

#### Questions/Design Decisions

* [ ] Should there be a `list(collectionName)` api method? issue #1
* [ ] Should there be a way to list all collections? (created explicitly or
  lazily) issue #3
