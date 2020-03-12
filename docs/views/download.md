---
id: download
title: Download response
---

A last helper in the response object is here to help you to render something from your callback route.
This helper allows you to return a file which will be download through the client browser.

The first value sent to the method can be either a path to a file, either the content of the file itself.
Again and like the two previous helper method, the download has a configuration object :

|Key|Type|Description|Default value|
|---|---|---|---|
|headers|object|The headers bound to the response|{ 'Content-Disposition': 'attachment' }|
|statusCode|number|The status code of the response|200|
|isFilename|bool|To know if the first parameter of the download function is a filename or the content of the file|true|
|sourceFilename|string|The path of the file to read it|If `isFilename` is true, it will take the first parameter as value|
|downloadFilename|string|The name of the file the user download|The value of `sourceFilename` if not set|
|readFileOptions|string|What is the right encoding format to read the file you want to download. It's the second parameter sent to readFile if you need more informations|'utf8'|

Unlike the previous helper which can be use without configuration, it is highly recommanded to setup your configuration here.

> Why ? Because the downloadFilename takes the value of the sourceFilename automatically if no value is set. If your sourceFilename is a path, it contains slashes, so we can't assure you the behavior of the browser

Something good to know is that the default configuration is loaded first, then the sourceFilename is set (automatically the first parameter is set) and then your configuration. So you can just specify a configuration and send null as the first parameter, it can work too.

Like every other response built-in helper, you can use it in your route callback method. Here are some examples :

```javascript

// The most basic usage
response.download('./my_files/foo.txt', {
  downloadFilename: 'your_foo_file.txt'
})

// If you have a content as a string variable and you want to make it download to your user
response.download('The content of the file downloaded', {
  isFilename: false,
  downloadFilename: 'file.txt'
})

// To illustrate what I said about the configuration which can be used alone
response.download(null, {
  isFilename: true, // useless because the default value is true but it's just to show with all the informations
  sourceFilename: './my/foo/file.bar',
  downloadFilename: 'file.txt',
  readFileOptions: { encoding: 'latin1' }// if your file is encoded with the format ISO-8859-1 for example
})
```

It's quite simple and it is a very useful shortcut for the download instead of coding it yourself.
It's the last built-in helper for the response provided by Cherry, but of course if you want to respond something else, you can still use the response object and do whatever you want with it. It's a classic [ServerResponse](https://nodejs.org/api/http.html#http_class_http_serverresponse) instance from Node.

So now, let's talk in the next chapter something more advanced... The plugins !
