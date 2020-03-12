---
id: json
title: Render a json payload
---

You want to create an API ? You need to return some json ? Yeah, you could do it by yourself with the response object, but what about circular JSON for example ?

Like for the HTML in the previous chapter, an helper is added to the response object, it works the same way, but unlike the HTML built-in process, you don't have any plugin to install for advance features.

Here are the options available in the configuration parameter :

|Key|Type|Description|Default value|
|---|---|---|---|
|headers|object|The headers bound to the response|{ 'Content-Type': 'application/json' }|
|statusCode|number|The status code of the response|200|
|serializer|function|A method call to format the values|null|
|indent|number|The number of spaces used by indentation|2|

For the serialization, the library [json-stringify-safe](https://www.npmjs.com/package/json-stringify-safe) is used.
If by any chance you already have a json string, you can still provide it to the method, but it won't be modified.

Here are some examples (to use in a route callback of course) :

```javascript
response.json({
  foo: 'bar'
})

/*
 * Returns a response with the status code 200 like this :
 * {
 *   "foo": "bar"
 * }
 */

 // -------

response.json([
  1,
  2,
  'foo'
], {
  ident: 4,
  statusCode: 400,
  headers: {
    'Content-Type': 'text/plain'
  }
})

/*
 * Returns a response with the status code 400 with the header "Content-Type" as "text/plain" like this :
 * [
 *     1,
 *     2,
 *     "foo"
 * ]
 */

 // -------

const jsonString = JSON.stringify({ test: 'Hello' })
response.json(jsonString)

/*
 * Returns a response with the status code 200 like this :
 * { "test": "Hello" }
 */

```

As you can see, there is nothing hard about it. Let's code your APIs !

> Future improvement possible : Add an option to replace the [Circular] wording, the library json-stringify-safe provides this feature some why not
