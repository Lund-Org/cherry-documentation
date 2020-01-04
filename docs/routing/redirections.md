---
id: redirections
title: Redirections
---

Let's talk about the last part the routing chapter.

In a website, we need to redirect our users. There are two kind of redirections :
- The one in the code which is kind of a technical redirection (like changing a page after the submit of a form)
- The one before executing the code, for SEO purposes. If you're familiar with Apache, it's like a htaccess


## Technical redirections

The first redirection is possible thanks to the method `redirect` in the response object like this :
`response.redirect('/my/redirect/url', 301)`

The first parameter is the url of the redirection, the second one is optionnal, it's the redirect HTTP code (by default 301).


## SEO redirections

Like the routes and the middlewares, it's a new thing to register in the Cherry options. Here is the format of the redirections :

|Option name|Optionnal|Value Type|Description|Default value|
|---|---|---|---|---|
|matchUrl|❌|string&vert;RegExp|The matching pattern to detect if we need to redirect|-|
|targetUrl|❌|string|The new route where we want to redirect|-|
|statusCode|❌|number|The HTTP code (which is between 300 and 310)|-|
|keepQueryString|✔|boolean|To know if we keep the query string (the get parameters) or no|false|


So here is an example when you register your redirections :

<!--DOCUSAURUS_CODE_TABS-->
<!-- main.js -->

```javascript
const Cherry = require('@lund-org/cherry')
// We get our route config
const routes = require('./config/routes.js')
// We get our redirection config
const redirections = require('./config/redirections.js')

const options = {
  servers: [
    { port: 80 }
  ],
  routes,
  redirections // as simple as the routes and middleware
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)

```


<!-- config/redirections.js -->

```javascript
module.exports = [
  {
    matchUrl: /^\/old-users\/(.*)/,
    targetUrl: '/users/$1',
    statusCode: 301
  }
]

```

<!--END_DOCUSAURUS_CODE_TABS-->

With only one redirection, all the matching paths beginning with `/old-users/` will redirect to `/users/xxx`.
Examples :
- /old-users/1 will redirect to /users/1
- /old-users/edit will redirect to /users/edit
- /old-users/?foo=bar will redirect to /users/ because the keepQueryString has not been defined (so it's false by default)

So it's just a regex to match the routes, and a regex to create the new url
