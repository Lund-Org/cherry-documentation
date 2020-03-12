---
id: html
title: Render a html view
---

The response object provided by the framework in the route callbacks has multiple built-in methods.
In this chapter, we will take a look to the HTML view rendering which is one of the main part of a web framework.

By default, and without plugin, you can only render raw html without variables inside it. It is useful it you build a static website.
You have to way to use the default html rendering :
- Using a raw html string passed to the method
- Using the file path of the view

Here are two examples for both of the cases :
```javascript
// Let's imagine you have 2 routes :

{
  type: 'ROUTE',
  method: ['GET'],
  path: '/html-raw',
  callback: (request, response) => {
    return res.html(
      '<div style="background: red; width: 500px; height: 500px;"></div>',
      { isRaw: true }
    )
  }
},
{
  type: 'ROUTE',
  method: ['GET'],
  name: 'html-from-file',
  path: '/html-file',
  callback: async (req, res) => {
    return res.html(
      path.join(__dirname, '/my-handlebar-view.html')
    )
  }
}
```

By default, the first parameter expected is a file path, but you have some configurations possible with the second parameter.
In the first example, you can see I specified the fact that I provided a raw html.

This configuration object may have more value than just specifying if it's a raw string or not.
|Key|Type|Description|Default value|
|---|---|---|---|
|headers|object|The headers bound to the response|{ 'Content-Type': 'text/html' }|
|statusCode|number|The status code of the response|200|
|isRaw|boolean|If what is provided is a path or a raw string|false|
|parameters|object|The variables to inject in the template|{}|

But... wait. There is a parameter key in the configuration. As I said earlier, we can't inject variables to static pages.
To get dynamic pages, you need to install a view rendering engine. Two plugins have been made to use the most used templating engine : [Handlebars](https://github.com/Lund-Org/cherry-handlebars-connector) and [Pug](https://github.com/Lund-Org/cherry-pug-connector).
Of course, you can create your own view plugin for another templating engine, you just have to follow the interface.

Let's focus on the Handlebar plugin, but the other ones works the same way.
First of all, you need to install the plugin :
```npm i @lund-org/cherry-handlebars-connector```

Once it's done, you need to inject it in Cherry like every plugins :
```javascript
const routes = require('./routes')
const Cherry = require('../../src/cherry')
const CherryHandlebarsConnector = require('@lund-org/cherry-handlebars-connector')

const options = {
  servers: [
    {
      port: 4000
    }
  ],
  plugins: [
    CherryHandlebarsConnector
  ],
  routes
}

const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

And.... it's done ! Now you can render an html file with the additionnal features provided by Handlebars like conditions, loops, variables...
For example, let's say one of our route callback looks like this :

```javascript
// In the route file

{
  ...
  callback: function (request, response) {
    return response.html(
      path.join(__dirname, '/my-handlebar-view.html'),
      {
        parameters: {
          foo: 'bar',
          routeName: req._route.name
        },
        viewEngine: 'utf8'
      }
    )
  }
  ...
}

```

As you can see, there is a new parameter "viewEngine". I didn't talk about it earlier because it is a parameter specific to the handlebars plugin. It is optionnal anyway.

And now, let's take a look to our html template :
```html
<p>Hi {{ foo }}</p>
<p>Here is an example using Handlebar thanks to the route : {{ routeName }}</p>
```

Because the parameters `foo` and `routeName` have been provided, it will be replaced and give the final html processed you expect.
Of course, you can use the tools from your templating engine to use a layout, some includes, etc.

One advice I can give is to store a view folder path in the `global` variable to easily render your views :

<!--DOCUSAURUS_CODE_TABS-->
<!-- File structure -->

```
Assuming this folder structure :

+--- index.js (the entrypoint of the project)
+--- views
|     \--- path
|         \--- to
|             \--- my
|                 \--- view.html
```
<!-- index.js (the entrypoint of the project) -->

```javascript
global.view_folder = path.join(__dirname, './views')
```

<!-- In the callback function -->

```javascript
// In your callback function
response.html(
  path.join(global.view_folder, './path/to/my/view.html')
)
```

<!--END_DOCUSAURUS_CODE_TABS-->


Another possibility if you use classes for your callbacks, you can make an abstract and do something like this :


<!--DOCUSAURUS_CODE_TABS-->
<!-- File structure -->

```
Assuming this folder structure :

+--- index.js (the entrypoint of the project)
+--- helpers
|     \--- AbstractController.js
+--- modules
|     \--- users
|         \--- controllers
|             \--- UserController.js
|         \--- views
|             \--- profile.html
```
<!-- AbstractController.js -->

```javascript
const path = require('path')

module.exports = class AbstractController {
  constructor (controllerDirname) {
    this.viewFolder = path.join(controllerDirname, '../views')
  }
}
```

<!-- UserController.js -->

```javascript
const path = require('path')
const AbstractController = require('../../../helpers/AbstractController')

module.exports = class UserController extends AbstractController {
  constructor () {
    super(__dirname)
  }

  index (request, response) {
    response.html(
      path.join(this.viewFolder, 'index.html')
    )
  }
}
```

<!--END_DOCUSAURUS_CODE_TABS-->

It is just silly examples, you can do what you want with Cherry so the folder structure is not fixed, but it's a good idea to have these little helpers to find your view folder easily.

So now, you will be able to create a basic website, static or not. But what if you want to create an API to have some Ajax in your pages ?
