---
id: basic
title: Basic routes
---

Binding public resources is useful, but we need more for a website which is not static.

The starting point of every request coming to Cherry is the routes. We've seen the public folder which has the biggest priority but the second thing checked if the file doesn't exist in a public folder is the configured route. Like every framework, we bind a path pattern to a process to execute in case of a match.

## Basic example

Imagine we want a classic MVC pattern, we can create a project which looks like that :

```
- config
  - routes.js
- modules
  - users
    - controllers
      - IndexController.js
    - models
      - User.js
    - views
      - index.html
      - edit.html
      - show.html
main.js

```

So let's imagine we want a classic [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) for our entity User. We want some routes, each routes are bind to a function and with a specific view. For the example, I created a folder named "models" with a "User.js" inside but we will not use it since we don't have a database and it's just to show a MVC pattern to sort your files. But again, Cherry is made to be used at your convenience.

Let's register our first route to show the list of the users :

<!--DOCUSAURUS_CODE_TABS-->
<!-- main.js -->

```javascript
const Cherry = require('@lund-org/cherry')
// We get our route config
const routes = require('./config/routes.js')
// We want templated html so I use the handlebar connector but you can use other connector
// Don't panic, we will talk about it later. Just assume it's a magic thing to inject cool stuff in an html view
const CherryHandlebarsConnector = require('@lund-org/cherry-handlebars-connector')

const options = {
  servers: [
    { port: 80 }
  ],
  plugins: [
    CherryHandlebarsConnector
  ],
  routes // We add it here to configure the Cherry server
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

<!-- config/routes.js -->

```javascript
const UserIndexController = require('../modules/users/IndexController')

module.exports = {
  // remember, it's the key for the public resources.
  // We don't want public resources for our example, so the key could be removed, but it's just a reminder
  publicRouter: [],
  router: [
    {
      type: 'ROUTE',
      path: '/users',
      // Here is a first example of how to do, in the comments bellow I will show you another way to do
      // Choose what you prefer !
      callback: (request, response) => {
        const ctrl = new UserIndexController()
        return ctrl.index(request, response)
      }
      /*
        // see the comments in the controller
        callback: UserIndexController.index
      */
    }
  ]
}
```

<!-- modules/users/IndexController -->

```javascript
const path = require('path')

class IndexController {
  index (request, response) {
    // Let's imagine you get from a database your users in a variable named users.
    // For our example, the list will be hard coded
    const users = [
      { id: 1, username: 'Foo' },
      { id: 2, username: 'Bar' }
    ]

    // Don't worry for the html render, we will talk about it later
    return response.html(
      path.join(__dirname, '../views/index.html'),
      {
        parameters: { users }
      }
    )
  }
}

module.exports = IndexController
/*
  // With the second example to register your route :
  module.exports = new IndexController()
*/
```

<!-- modules/views/index.html -->

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- We don't care for the example -->
  </head>
  <body>
    <!-- We display the list of users -->
    <!-- If you're not familiar with Handlebar, take a look here : https://handlebarsjs.com/ -->
    <table>
      <!-- We iterate on each users to display a line by user with its identifier and username -->
      {{#each users}}
        <tr>
          <td>{{this.id}}</td>
          <td>{{this.username}}</td>
        </tr>
      {{/each}}
    </table>
  </body>
</html>

```

<!--END_DOCUSAURUS_CODE_TABS-->

And with just these little code lines, we have a working list of our users. But because we are working on the routes, let's focus on it.
My route is declared with these few lines :
```javascript
{
  type: 'ROUTE',
  path: '/users',
  callback: (request, response) => {
    const ctrl = new UserIndexController()
    return ctrl.index(request, response)
  }
}

```

## Route types

In Cherry, we have 3 types of routes.
- The first one, we've already seen it in the previous chapter, is `PUBLIC_ROUTE_PUBLIC_FOLDER`. It's a type a little special because it's for the public routes (under the key `publicRouter` in the routes object of the Cherry config).
- The second one is the `ROUTE` which defines a single route like we did above. The options which can be join to the type are the following :

|Option name|Optionnal|Value Type|Description|Default value|
|---|---|---|---|---|
|path|❌|string&#124;regex|The format of the url path|-|
|callback|❌|function|The function to execute when the url is reached|-|
|method|✔|array<string>|The HTTP methods which should trigger the callback if the url match|`['*']`|
|name|✔|string|The name of the route to identify it. Can be useful in hooks (see below)|`no-name-route-`|
|rules|✔|object|The pattern for the parameters in the routes. The key is the parameter name, the value the regex to match|`{}`|
|middlewares|✔|array<string>|The list of the middleware to apply (see later)|`[]`|

- The third type of route will be explain in the next chapter, it's `ROUTE_CONTEXT` which represents a context


As you've seen in the parameters that a route can take, some options can be useful for us. We will ignore `name` and `middlewares` for now because it's not important for our use-case

## An entire CRUD

### The list route

Let's continue our example to create our CRUD.
I will not code the html views because it's not the subject and is not the most important part.
Remember, our route to list every users looks like this :
```javascript
{
  type: 'ROUTE',
  path: '/users',
  callback: (request, response) => {
    const ctrl = new UserIndexController()
    return ctrl.index(request, response)
  }
}

```

In fact, it should be only when we target `/users` with the GET method. Let's add this rule to the route thanks to `method: ['GET']`.

It will result with :
```javascript
{
  type: 'ROUTE',
  path: '/users',
  method: ['GET'],
  callback: (request, response) => {
    const ctrl = new UserIndexController()
    return ctrl.index(request, response)
  }
}

```

Fantastic ! Now, we need a route to create an user, a route to see one specific user (it's profile page for example), a route to update an user and finally a route to delete an user.

### The create route

Because it's easy to do, I will directly give you the route to create an user, because it looks a lot like the index route. The important thing in this example is in the controller : how to get the data of the body of the request :

<!--DOCUSAURUS_CODE_TABS-->
<!-- main.js -->

```javascript
// Nothing changed here, this is the last time I put this file in the example, assume in the next ones the file doesn't change

const Cherry = require('@lund-org/cherry')
const routes = require('./config/routes.js')
const CherryHandlebarsConnector = require('@lund-org/cherry-handlebars-connector')

const options = {
  servers: [
    { port: 80 }
  ],
  plugins: [
    CherryHandlebarsConnector
  ],
  routes // We add it here to configure the Cherry server
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

<!-- config/routes.js -->

```javascript
const UserIndexController = require('../modules/users/IndexController')

module.exports = {
  // I deleted the publicRouter key to simplify the code
  router: [
    {
      type: 'ROUTE',
      path: '/users',
      callback: (request, response) => {
        const ctrl = new UserIndexController()
        return ctrl.index(request, response)
      },
      method: ['GET']
    },
    {
      type: 'ROUTE',
      path: '/users',
      callback: (request, response) => {
        const ctrl = new UserIndexController()
        return ctrl.create(request, response)
      },
      method: ['POST']
    }
  ]
}
```

<!-- modules/users/IndexController -->

```javascript
const path = require('path')
const User = require('../models/User')

class IndexController {
  index (request, response) {
    // see previous example, it's not the part interesting here
  },
  create (request, response) {
    // In the request variable, you can find a lot of informations :
    // - request.body => The body as a string. Ex : '{ "foo": "bar" }'
    // - request.params => The body interpreted and the query parameters in an object. Ex : { foo: "bar" }
    // - request._route => The route which matched. Do not modify this object, just read it, or you might break your app. Wil rarely be useful in a controller

    // Here is an example of what you can do in your create route. The User class doesn't exist, remember it's just an example
    const user = new User(request.params.user)
    if (user.save()) {
      response.redirect(`/users/${user.id}`)
    } else {
      // the form to create the user, with the data previously filled
      return response.html(
        path.join(__dirname, '../views/form.html'),
        {
          parameters: { user }
        }
      )
    }
  }
}

module.exports = IndexController

```

<!--END_DOCUSAURUS_CODE_TABS-->

The example is not accurate because we don't have a view for the form before calling the create action, but you get the idea and how it works.

### The profile/delete/update route

You should ask, why do I group the 3 actions together ? The answer is pretty simple : Because it uses the route parameters which is the last thing to show to you.
