# kvplus-js
Specs for a simple Key/Value (plus Secondary Indexes) store API, for Node.js/Javascript

## Contents

1. `get()`
2. `put()`
3. `del()`

### CRUD

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
