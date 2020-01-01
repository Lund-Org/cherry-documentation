---
id: context
title: Route contexts
---

The context route is a tool to help you to build routes easily.

Remember our routes from the previous chapter with a REST CRUD API for an user entity :

```javascript
// routes.js
const UserIndexController = require('../modules/users/IndexController')

module.exports = {
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
    },
    {
      type: 'ROUTE',
      path: '/users/:id',
      callback: (request, response) => {
        const ctrl = new UserIndexController()
        return ctrl.show(request, response)
      },
      method: ['GET'],
      rules: { id: /\d+/ }
    },
    {
      type: 'ROUTE',
      path: '/users/:id',
      callback: (request, response) => {
        const ctrl = new UserIndexController()
        return ctrl.update(request, response)
      },
      method: ['PATCH'],
      rules: { id: /\d+/ }
    },
    {
      type: 'ROUTE',
      path: '/users/:id',
      callback: (request, response) => {
        const ctrl = new UserIndexController()
        return ctrl.delete(request, response)
      },
      method: ['DELETE'],
      rules: { id: /\d+/ }
    }
  ]
}

```

There are two things annoying in this example.
- Imagine a developer of your team add a new route for the user resource, he can add it at the end of the route file (because maybe he doesn't follow good practice or it's a test route which becomes finally a real route). If you remake your controller, it would be easier to locate all the routes as one block instead of scrolling or searching in the file
- It's annoying to duplicate informations like the `/users/` path for each route

Here is an example to explain the second point above more clearly.
Imagine you have an administration with a lot of paths beginning with `/admin/`, using the GET method. Each routes need to have a name prefixed with `"admin-"`. Finally, every route should have a security check (if the user is connected and has the permission) achieved thanks to a middlevare. You don't want to duplicate all these informations for each route. Here comes the context to help you.

|Option name|Optionnal|Value Type|Description|Default value|
|---|---|---|---|---|
|collection|❌|array&lt;object&gt;|The list of the routes which will be in the context|-|
|name|✔|string|The prefix which will be append to the routes in the collection array|`no-name-route-`|
|method|✔|array&lt;string&gt;|The HTTP methods which should trigger the callbacks for the routes in the collection. It becomes the default value for the routes without methods but will be override if a method is declared in the routes|`null`|
|path|✔|string|The prefix for every route paths for the routes in the collection|`/`|
|middlewares|✔|array&lt;string&gt;|The list of the middleware to apply for each routes in the collection ([see later](routing/middlewares.md))|`[]`|
|rules|✔|object|The pattern for the parameters in the context path. The key is the parameter name, the value the regex to match|`{}`|


With all these information, here what our user REST CRUD API will look like :

```javascript
// routes.js
const UserIndexController = require('../modules/users/IndexController')

module.exports = {
  router: [
    {
      type: 'ROUTE_CONTEXT',
      path: '/users',
      name: 'users-',
      method: ['GET'],
      collection: [
        {
          type: 'ROUTE',
          path: '/',// the route become / since we have /users as a prefix
          callback: (request, response) => {
            const ctrl = new UserIndexController()
            return ctrl.index(request, response)
          }
          // no need to define the GET method
        },
        {
          type: 'ROUTE',
          path: '/',
          callback: (request, response) => {
            const ctrl = new UserIndexController()
            return ctrl.create(request, response)
          },
          method: ['POST']
        },
        {
          type: 'ROUTE',
          path: '/:id',
          callback: (request, response) => {
            const ctrl = new UserIndexController()
            return ctrl.show(request, response)
          },
          rules: { id: /\d+/ }
        },
        {
          type: 'ROUTE',
          path: '/:id',
          callback: (request, response) => {
            const ctrl = new UserIndexController()
            return ctrl.update(request, response)
          },
          method: ['PATCH'],
          rules: { id: /\d+/ }
        },
        {
          type: 'ROUTE',
          path: '/:id',
          callback: (request, response) => {
            const ctrl = new UserIndexController()
            return ctrl.delete(request, response)
          },
          method: ['DELETE'],
          rules: { id: /\d+/ }
        }
      ]
    }
  ]
}

```

This code doesn't show all the potential of the context, but it shows the syntax. When you'll learn the next chapter about the middleware, and with the example of the administration, you will easily understand the power of the contexts.

Of course, you can chain them. It means we could add the resources with the id in the route parameter in a context which manage it. If we want to provide security on the update/delete route of our user entity (to check that the user updated is the one connected), we are able to set a middleware for a context and put these route in it. And if you add a route which requires this same security, you can add it in the block without thinking about it, you will have the same behaviour for all the routes in the context.

It's all about reliability, not repeating yourself, isolating the similar routes in a same block to keep all of it ordered. The next chapter will explain the middlewares and you will maybe see how useful the context can be.
