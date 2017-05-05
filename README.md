# kvplus-js
Specs for a simple Key/Value (plus Secondary Indexes) store API, for Node.js/Javascript

**Current Spec version:** `v.0.1.0` (see [CHANGELOG.md](CHANGELOG.md))

### Contents

* [Motivation](#motivation)
* [Store API](#store-api)
  * [Creating a Store Instance](#creating-a-store-instance)
  * [Initializing a Collection](#initializing-a-collection)
* [Store CRUD API](#store-crud-api)
  * [`list()`](#list)
  * [`exists()`](#exists)
  * [`get()`](#get)
  * [`put()`](#put)
  * [`remove()`](#remove)
* [Store Secondary Index API](#store-secondary-index-api)
  * [`createIndex()`](#createindex)
  * [`findBy()`](#findby)
* [Implementations](#implementations)

### Motivation

This K/V API was initially created to provide a pluggable persistence interface
for the [`oidc-op`](https://github.com/anvilresearch/oidc-op) OpenID Connect
Identity Provider library (to store user accounts, client registrations, access
tokens, and so on).

#### Why not just use [`abstract-blob-store`](https://github.com/maxogden/abstract-blob-store)?

There are several design motivations for K/V Plus and differences from the
Abstract Blob Store (ABS) API:

1. **Collections.** ABS (like
  [`leveldown`](https://github.com/rvagg/abstract-leveldown)) has a flat
  keyspace. We explicitly wanted to organize keys in bucket-like named
  Collections.
2. **API for reading/writing smaller objects.** ABS is designed for reading and
  writing potentially large binary objects, hence its interface is stream-based.
  However, for most small objects (and JSON documents), read and write streams
  is overkill, and a Promise-based API makes more sense.
3. **Listing keys.** Key/Value stores are great, but being able to list the keys
  in a given collection is incredibly important (just ask long-time
  [Riak](https://github.com/basho/riak/) users).
4. **Secondary Indexes.** Similarly, the addition of simple secondary index
  capability (for example, being able to find users by email in addition to
  fetching them by user id) makes K/V stores significantly more useful for most
  dev purposes.

### Store API

#### Creating a Store Instance

Usage:

```js
const options = {
  path: './db',
  serialize: (obj) => { return JSON.stringify(obj) }
}

const store = new KVPlusStore(options)
```

##### Constructor Options

* `path` - (string) For filesystem based stores, specifies the directory
  where the store will be created. *Optional.*
* `serialize` - (Function) If storing objects other than strings, provide a
  serialization function that will turn an object into a string representation.
  Can also be provided for the whole store (in the store constructor), or
  overridden for each collection.

### Initializing a Collection

`Promise createCollection (string collectionName, Object options = {})`

Creates/initializes a collection in the store. If the backend storage mechanism
does not have any need for initializing a collection (such as an in-memory
store), the implementation should do something like `return Promise.resolve()`.

*Important:* This operation should be *idempotent* -- if the collection already
exists, it should result in a no-op.
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

### Store CRUD API

#### list()

`Promise<Array<string>> list (string collectionName, Object options = {})`

Returns a list of keys in a given collection.

Usage:

```js
store.put('users', 'user1', { name: 'Bob' })

  .then(() => store.put('users', 'user2', { name: 'Alice' }))

  .then(() => store.list('users'))

  .then(result => {
    // result -> [ 'user1', 'user2' ]
  })
```

#### exists()

`Promise<boolean> exists (string collectionName, string key)`

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

`Promise<Object> get (string collectionName, string key, Object options = {})`

Retrieves the serialized object from the store, from the given collection.

Usage:

```js
store.get('users', 'u1')
  .then(retrievedUser => {
    console.log(retrievedUser)  // user object with key 'u1' retrieved
  })
```

#### put()

`Promise<Object> put (string collectionName, string key, Object data, Object options = {})`

Writes an object to the store, and returns the object that was written. Note:
Serves both as an Insert and an Update operation (if an object already exists
for that key, in that collection, it gets overwritten.)

Usage:

```js
let user = { name: 'Alice' }

store.put('users', 'u2', user)
  .then(storedUser => {
    assert(storedUser.name === 'Alice')
  })
```

### remove()

`Promise<Boolean> remove (string collectionName, string key)`

Attempts to remove an object for a given collection and key. If the object
existed and the delete succeeded, resolves to `true`. If the object did not
exist, resolves to `false`.

Usage:

```js
store.remove('users', 'u2')
  .then(() => {
    // user deleted. In this case, we don't care about whether the user actually
    // existed, so we're not going to check the boolean result of the operation
  })
```

### Store Secondary Index API

#### createIndex()

`Promise createIndex (string collectionName, string property, Object options = {})`

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
* LocalStorage KVPlus store, for in-browser use.

#### Questions/Design Decisions

* [ ] Should there be a way to list all collections? (created explicitly or
  lazily) issue #3
