---
id: middlewares
title: Middlewares
---

First of all, let's define what is a middleware. This is a little component that will be executed between the creation of the request and the execution of the callback bound to your route. It's very useful to add some checks (security, permissions...) or to process data (to prepare it for the callback for example).

Here are the datas to describe a middleware :

|Option name|Optionnal|Value Type|Description|Default value|
|---|---|---|---|---|
|name|❌|string|The name of the middleware to register it in the routes|-|
|callback|❌|function(next, request, response)|The method executed when the middleware is triggered|-|

As you can see, the callback doesn't have just the request and the response object, there is the `next` parameter which is the next entity to execute. It can be a middleware (because you can chain it) or the route callback. If you doesn't respond to the client request (thanks to the response object) or if you don't execute the next action of the callback chain, the request will be pending.

Let's imagine an user which is connected to your website, and you put in its cookies something to recognize him like a hash. We want to get the real user behind this hash and not call a method which do it in every controller. The middleware is the perfect tool to achieve it.

First of all, let's declare a middleware without writing the content of the callback

```javascript
// middlewares.js

module.exports = [
  {
    name: 'getConnectedUser'
    callback: (next, req, res) => {
      // nothing for now
      next.resolve(req, res)
    }
  }
]

```

Now we need to register our middleware in the Cherry options, under the key... middlewares. Of course.

```javascript
// main.js
const Cherry = require('@lund-org/cherry')
// We get our route config
const routes = require('./config/routes.js')
// We get our middleware config
const middlewares = require('./config/middlewares.js')

const options = {
  servers: [
    { port: 80 }
  ],
  routes,
  middlewares // as simple as the routes
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

So now, Cherry will know what is the 'getConnectedUser' middleware. In your route configuration, you can now use it.

```javascript
// routes.js

module.exports = {
  router: [
    {
      type: 'ROUTE',
      path: '/my/route',
      callback: (request, response) => {
        response.end('<div>It works !</div>')
      },
      middlewares: ['getConnectedUser']
    }
  ]
}
```

Just with it, the middleware will be executed each time the route is reached. So now, let's code what we need in the middleware, I will just get the cookie value I need and code like if we had an entire app (the functions used don't exist, but it's for the example) :

```javascript
// middlewares.js

// Imagine an user repository which allows you to retrieve the users from a database
const UserRepository = require('../src/repositories/UserRepository')

module.exports = [
  {
    name: 'getConnectedUser',
    callback: (next, request, response) => {
      request.user = null
      if (typeof request.headers.cookie !== 'undefined') {
        // We get the value of the cookie 'my-super-hash' thanks to a regex
        // It is under the format : "CookieName1=Value1; CookieName2=Value2"
        const userHash = request.headers.cookie.match(/my-super-hash=([A-Za-z0-9]+)/)

        // if there are no match, it means the cookie doesn't exist
        if (userHash) {
          const user = UserRepository.findByHash(userHash[1])

          // if the user is found, we put it in the request
          if (user) {
            request.user = user
          }
        }
      }

      next.resolve(request, response)
    }
  }
]

```

And now, our registered route can access to the current connected user entity by getting it thanks to `request.user`
Because a request is just living during one call, you can put datas inside, just remember to keep it clean. You can for example set your datas in an object to have everything at the same place.

Our example is just to get a data easily for several routes, if we wanted to have a security, like an area available only for connected user (the user account management for example), we could have throw an error, or returned a 404 page thanks to the `response` variable.

And now, you can understand how much the context routes can be useful with the middlewares. It avoids to put in on every route, you just have to set it on the context and every routes inside will use it.
