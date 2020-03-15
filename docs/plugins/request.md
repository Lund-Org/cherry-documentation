---
id: request
title: Request Plugins
---

Another core plugin is the one with the identifier `RequestEngine`.
A default behavior exists to handle the requests. If you send a payload as a json, it will be parsed otherwise, it will be considered as a querystring. These POST datas and the GET datas are aggregate in a `params` attribute of the request to access easily to it in your route callback.

If you want to handle a x-www-form-urlencoded form, it is not native to Cherry and you need to create a plugin for it. Here is the interface to follow for a request plugin :

```javascript
class MyRequestPlugin {
  // The method called with the "onData" event of the request
  // It gives you chunk of data you can aggregate in the chunks array
  aggregateData (promiseCallbacks, chunks, data) {
    /*
      - promiseCallbacks is an object like this : { resolve, reject }.
      It allows to finish the request with a success or an error. I don't see a reason to use it, but you can
      - chunks is an array of all the chunk you added with this method
      - data is a chunk of the payload you received
    */
  }

  // Once you got all the payload of you request, if no error happened, you can process the entire data
  // The default process is to parse the payload and store the data in the request with the attributes "params" as an object
  // It is triggered on the event onEnd of the request
  finalizeRequest (promiseCallbacks, chunks) {
    /*
      - promiseCallbacks is an object like this : { resolve, reject }.
      It allows to finish the request with a success or an error. The default method resolve "this" which is the request object itself
      - chunks is an array of all the chunk you added with this method
    */
  }

  // If an error happened while the request is processed, this callback is triggered
  errorManagement (promiseCallbacks, error) {
    /*
      - promiseCallbacks is an object like this : { resolve, reject }.
      It allows to finish the request with a success or an error. The default method resolve "this" which is the request object itself
      - error is an error received through the event onError of the request (request abort for example)
    */
  }
}
```

If you want to create a simple API or a simple website with just classic forms, the default request plugin can be used.

In an other hand, if you want to upload file or use a x-www-form-urlencoded form, this request manager is not enough.

> An issue in the Github repository exist to provide this core plugin, but it needs to be done.

As said previously, all the datas are set in a `params` attribute of the request object. So imagine you have this request coming :

```
POST /my/route?foo=bar&limit=1
Body :
{ "foo": "nobar", "page": 12 }
```

You can find all these datas like this :

```javascript
function routeCallback (request, response) {
  console.log(request.params)
  /*
   * {
   *   foo: 'nobar',
   *   page: 12,
   *   limit: 1
   * }
   */
}
```

> As you can see, the payload data has more priority than the query parameter. So the "foo=bar" in the url is lost. You can still find it through the request object with a parsing (with the url module of node) and the url provided by request.url

So now, we've seen the core plugins. But what about your custom plugins you can add to your app, or even plugins you can develop and share in the open source world ?

Let's check in the next chapter.
