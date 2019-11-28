---
id: public
title: Public resources
---

# Introduction

First of all, most of the routes of a website are simply path for resources like images, styles or scripts. We don't want to create a route or a regex to target a controller which will return the content of the file to the browser, manage it automatically is a base for every websites.

Some frameworks just have a public folder where you can put everything and the path will be resolve. With Cherry, the goal was to allow multiple public spaces. Based on some experiments in several job, it appears that sometimes you can have a public folder for your resources, sometimes you have a folder with you javascript apps builded, sometimes you want to target a S3 bucket (not implemented yet).

So to achieve it, when you register your route, you can provide the public folder you want to check for each call which doesn't match with the route of your app. It allows you to define your own structure of the app. Feel free to create a public folder like every other framework or to sort your files in another way.

# The public resource route type

## A basic example

With Cherry, when you create your app, you will register your routes. In the following example, imagine the routes in a separated file :

```javascript

// main.js
const Cherry = require('@lund-org/cherry')
const routes = require('./config/routes.js')

const options = {
  servers: [
    { port: 3000 }
  ],
  routes// <-- our routes are added here
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)

```

```javascript

// config/routes.js
const path = require('path')

module.exports = {
    publicRouter: [
      {
        type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
        path: path.join(__dirname, '../public')
      }
    ]
  }
}

```

This example gives us the following structure of files :
```

|- main.js
|- config
|   |- routes.js
|- public
    |- index.html
    |- my-image.png

```

Remember, Cherry doesn't force you to follow a specific pattern to sort your files. You can follow the **Best Practices** but it's our pattern like anything else, there is no best pattern since it's sorted, it's fine.

Let's imagine now we want a static website. I have my landing page in the public folder (the index.html). Here is what the index.html looks like :

```html

<!DOCTYPE>
<html>
  <head>
    <meta charset="UTF-8" />
  </head>
  <body>
    <img src="/my-image.png" alt="My landing page image" />
  </body>
</html>

```

If I start my app, what will be the routes available ?

Since we only registered the public folder, everything under it will be available. So to access my landing page, I need to reach : `http://my-website.com/index.html`
As you've seen, the html requires an image which is in our public folder too, so it will automatically be found too

But what happens if I have 2 public folders ? Well, it will check in the first one if the resource exists and if yes, it will return it. In the other case, it will process the other directory. But we can have an issue :

```javascript

// config/routes.js
const path = require('path')

module.exports = {
    publicRouter: [
      {
        type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
        path: path.join(__dirname, '../public1')
      },
      {
        type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
        path: path.join(__dirname, '../public2')
      },
      {
        type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
        path: path.join(__dirname, '../public3')
      }
    ]
  }
}

```

Imagine this route file, nothing crazy, just three folder registered. And because some reasons, I want to make the resources of the public3 checked first before the other folders. How can I achieve it ?

## A priority for each folder

It's pretty easy. By default, a public route will have a priority of 0, which means that every public routes have the same weight and it will be sorted depending the order of the registration in your route file.

Three choices are available for you to achieve the previous use case.


### Method 1 : You want to specify a priority for the public3 folder only

For this, just add the priority attribute. Remember, by default it's 0 so to be checked first, just put a negative number:

```javascript

...
{
  type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
  path: path.join(__dirname, '../public3'),
  priority: -12
}
...

```

### Method 2 : You want to specify the order for every folder, which gives you the hand on everything


```javascript

...
[
  {
    type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
    path: path.join(__dirname, '../public1'),
    priority: 2
  },
  {
    type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
    path: path.join(__dirname, '../public2'),
    priority: 3
  },
  {
    type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
    path: path.join(__dirname, '../public3'),
    priority: 1
  }
]
...

```

The risk is to forget the priority attribute when adding a new public folder, so it will have the priority 0 and will be the first folder checked.

### Method 3 : Not used the priority attribute and sort it directly in the route array


```javascript

...
[
  {
    type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
    path: path.join(__dirname, '../public3')
  },
  {
    type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
    path: path.join(__dirname, '../public1')
  },
  {
    type: 'PUBLIC_ROUTE_PUBLIC_FOLDER',
    path: path.join(__dirname, '../public2')
  }
]
...

```

The biggest risk here is if you don't have everything in only one file. It may happen, if your app grows, that you have your routes in multiple files and you aggregate it in one array. What is the final order ? You will never be sure. That's why it is better to specify the priority attribute if you need to.


Once again, most of the time, you will not care about file path conflicts, because you will not have the same directory tree between your public folder registered and your file required. And if you have the same directory tree, does it really matter to have multiple public folder ? Well, it's up to you and depends of your project, but I suggest you to avoid the third method above if your public resources are an aggregate of some files.

