---
id: server-creation
title: Create a server
---

Now, you have Cherry installed in your dependencies. To start with the framework, we'll need a HTTP server to receive client requests and to give them responses.

## Instanciate a server

To load your dependency installed previously, like any other npm module, you need to import it :

```javascript
const Cherry = require('@lund-org/cherry')
```

Or if you use Typescript, Babel or a Node.js version which manage the ES6 import/export :

```javascript
import Cherry from '@lund-org/cherry'
```

With your Cherry class, you can now instanciate a Cherry server. The basic case will be with this configuration :

```javascript
const options = {
  servers: [
    { port: 80 }
  ]
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

As you can see, Cherry takes an option object to be configured, here it describes the server port of our instance.

And.... it's done. I mean almost, if you try to only use this code, it will not work because a HTTP server with no route is useless, so you will get an error, but it's the next topic of the documentation.

Of course to run your code, you just have to execute this :


```bash
$ node your_file.js
```

## Instanciate multiple servers

But sometimes you need to bind multiple ports, with multiple instances. For example, the previous code was an instance of server bound to the default HTTP port. But how can we manage the HTTP and the HTTPS at the same time ?
Here is the answer :

```javascript
const options = {
  servers: [
    { port: 80 },
    {
      port: 443,
      https: true,
      httpsOptions: {
        key: fs.readFileSync('/path/to/the/key.pem')),
        cert: fs.readFileSync('/path/to/the/cert.pem'))
      }
    }
  ]
}
const cherry = new Cherry()
cherry.configure(options)
cherry.start(options)
```

This will create 2 identical servers, one managing the HTTP and one managing the HTTPS. I used the default ports but you're free to change the values of course.

Let's see now what we can do to use our Cherry application.
