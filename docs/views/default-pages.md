---
id: default-pages
title: Default error pages 
---

When a 404 error or a 500 error occured, default pages are rendered to the user.
But what if you want to provide a custom page ?

An option in the object provided to the Cherry Instance is available for you to provide your own pages.
This option is like this :

```
const options = {
  ...,
  defaultPages: {
    clientErrorPage: <func(Request, Response)>,
    serverErrorPage: <func(Request, Response)>
  }
  ...
}

```

You don't have to provide both of the keys `clientErrorPage` or `serverErrorPage`, you can if you want provide one page and leave the default one for the other one.

Because the default page manager is in the cherry Object, you can render a 404 or a 500 page in your route callback by using `request.cherry.defaultErrorPageConfigurator.manager`. You will have access to the methods `clientErrorPage` and `serverErrorPage`. Here is an example :

<!--DOCUSAURUS_CODE_TABS-->
<!-- main.js -->

```javascript
// Nothing changed here, this is the last time I put this file in the example, assume in the next ones the file doesn't change

const Cherry = require('@lund-org/cherry')
const routes = require('./config/routes.js')

const options = {
  servers: [
    { port: 80 }
  ],
  defaultPages: {
    clientErrorPage: (request, response) => {
      return response.html('<p>404 Error</p>', { statusCode: 404, isRaw: true })
    }
  },
  routes // We add it here to configure the Cherry server
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

<!-- config/routes.js -->

```javascript
module.exports = {
  router: [
    {
      type: 'ROUTE',
      path: '/',
      callback: (request, response) => {
        // Here is the example to return the 404 page :
        return request.cherry.defaultErrorPageConfigurator.manager.clientErrorPage(request, response)
      },
      method: ['GET']
    }
  ]
}
```

<!--END_DOCUSAURUS_CODE_TABS-->

The clientErrorPage method is triggered automatically when no matching routes are found

The serverErrorPage method is triggered automatically when an exception is thrown and not catch
