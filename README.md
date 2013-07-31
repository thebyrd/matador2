# Matador 2
At Medium, we started with an [MVC framework](https://github.com/Obvious/matador), but quickly found that it didn't suite our needs. We made modifications to it to the point where it no longer resembles the framework it claims to use. This project is an atttempt at creating a framework that resembles the way we write web applications at Medium.

## Installation

### CLI
```
$ npm install matador2 -g
```

### New Application
```
$ matador2 init my-app
$ cd my-app && npm install
```

### Start the App
```
$ node server
```

## Nodes
Instead of spreading application logic across models and controllers, matador2 breaks all units of work into nodes. Nodes are just functions specified in a file in the `/nodes` directory.

For example, `createUser` and `getUserById` are nodes defined below:
```javascript
module.exports = {
  createUser: function (db, body) {
    return db.putItem('users', {
      id: body.email,
      name: body.name
    }).execute()
  },
  getUserById: function (db, id) {
    return db.getItem('users', {id: id}).execute()
  },
  getAllUsers: function (db) {
    return db.newQueryBuilder('users')
    .setHashKey('column', '@')
    .execute()
  }
}

```

## Handlers
Handlers are methods mapped to routes that specify how nodes should be used to return requests. They differ from controllers because there shouldn't be any logic outside of the [Shepherd Builder DSL](https://github.com/Obvious/shepherd#building-nodes).

If the `routes.json` file looks like:

```javascript
{
  "/users/:email": {
    "get": "users.show"
  },
  "/users": {
    "get": "users.index",
    "post": "users.create"
  }
}

```

Then GET requests to `/` and `/users` will be handled by the `index` method in `/handlers/users.js` which might look something like:

```javascript

module.exports = {
  index: function (builder) {
    return builder
      .builds('app.db')
      .builds('getAllUsers')
        .using('app.db', 'req.body')
      .respond('views.users.index', 'getAllUsers')
  },
  show: function (builder) {
    return builder
      .builds('app.db')
      .builds('getUserById')
        .using('app.db', {'id': 'req.params.email'})
      .respond('views.users.show', 'getUserById')
  },
  create: function (builder) {
    return builder
      .builds('app.db')
      .builds('createUser')
        .using('app.db', 'req.body')
      .redirect('/')
  }
}

```

The one parameter for every handler is a Shepherd builder. However, Matador2 adds two methods to the api:
* `respond(template, nodeName)` will render `template` using the result of `nodeName`. If the query parameter `apiv` is specified then a the result of `nodeName` will be sent to the client as json in the form `{success: true, payload: <nodeName result>}`.
* `redirect(path)` will redirct to `path` or if `path` is the name of a node built in the builder it will redirect to its result.

You'll also notice that there are a couple of "global" graph nodes that can be used handlers:
* `req` is the Request that user is handling. It is identical to the express request variable
  - some common attributes of `req` include `req.body` (the request body), `req.params` (the parameter in the route path), `req.query` (the query parameter), & `req.session` (the current session).
* `app` is the matador2 application and holds data about middlewear and external dependencies.
  - An example of an external dependency is `app.db` which is an instance of [dynamite](https://github.com/Obvious/dynamite)
  - Additionally, there is also `app.cdn` which is an instance of [knox](https://github.com/learnboost/knox)





