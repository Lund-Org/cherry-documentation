---
id: adding-routes
title: Adding routes
---

Now we have an awesome server which is running with Cherry. And like every HTTP(s) server, we need routes to match the request and a process.

## Adding a basic route

### An app without routes

You already discovered the `servers` key in the object of configuration. Let's remind what is the code we have :


```javascript
const Cherry = require('@lund-org/cherry')

const options = {
  servers: [
    { port: 80 }
  ]
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

To avoid port collision with the other application you might have on your computer, I will use a server configured for the port 3000 like every JS app documentation, which is kind of a "dev port".


### A quick explanation of the routes

To add routes, we need a new option which is.... `routes` obviously. Cherry can manage multiple types of route, you will see that in the [Routing chapter](../routing/public.md) of the documentation.

In this getting started, we will configure our application with a simple route `/hello-world`. This route needs at least three datas :
- The `type` of route which is in our case "ROUTE"
- The `path` to know what are the requests which need to match
- The `callback` which is -as you probably guessed- the function executed when the path match

The configured routes need to be under the key `router`. For this example, I will not go more deeply into the basic route configuration, just trust me for now.

So if you add what we learned :

```javascript
const Cherry = require('@lund-org/cherry')

const options = {
  servers: [
    { port: 3000 }
  ],
  routes: {
    router: [
      {
        type: 'ROUTE',
        path: '/hello-world',
        callback: (request, response) => {
          console.log('I was here !')
          return 'hello world'
        }
      }
    ]
  }
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

### An app with routes

Now with our application, we can try 2 things.

First of all, the obvious one is to visit [/hello-world](http://localhost:3000/hello-world). When we reach this url, we have our `console.log` in the terminal (each time we refresh, a new log is printed). On the page, you can see the text we returned in the callback method.

The other thing I wanted to show you is if you try to reach the [root of your application](http://localhost:3000). We didn't configure this root, but a page appears. This is the default 404 page that you can configure (see the [Views & Server response chapter](../views/default-pages.md))

Of course, this is an example, you might sort you routes in another file and import it in your configuration to have a code cleaner.
