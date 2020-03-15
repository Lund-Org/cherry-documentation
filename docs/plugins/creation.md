---
id: creation
title: Create your own plugins
---

A plugin, as you've seen previously, is just a class. This class just need one thing : an unique identifier which can be retrieve with the static method `getIdentifier`.

This class has a very simple role : abstract something or connect cherry to an existing library

Let's imagine you want a new plugin, its role is to cache the response of a route and use the cache if it exists. You could create an helper which use redis for example and you could include this file to every controller you have. Your code would look like that :

<!--DOCUSAURUS_CODE_TABS-->
<!-- File structure -->

```
Assuming this folder structure :

+--- helpers
|     \--- cacheHelper.js
+--- modules
|     \--- users
|         \--- controllers
|             \--- IndexUserController.js
|     \--- articles
|         \--- controllers
|             \--- IndexArticleController.js
```
<!-- IndexUserController.js -->

```javascript
const cacheHelper = require('../../../helpers/cacheHelper')

module.exports = {
  myRoute (request, response) {
    if (cacheHelper.hasCache('user-index')) {
      return cacheHelper.getCache('user-index')
    }

    // do things

    const result = response.html('my/view.html')
    cacheHelper.cache(response)
    return result
  }
}
```

<!-- IndexArticleController.js -->

```javascript
const cacheHelper = require('../../../helpers/cacheHelper')

module.exports = {
  myRoute (request, response) {
    if (cacheHelper.hasCache('article-index')) {
      return cacheHelper.getCache('article-index')
    }

    // do things

    const result = response.html('my/view.html')
    cacheHelper.cache(response)
    return result
  }
}
```

<!--END_DOCUSAURUS_CODE_TABS-->

It's a silly example, but the issue here is to repeat the code, the inclusion, etc. You repeat a lot of code. You can have less line of code with the inheritence but it's still not beautiful.

With a plugin, first of all you can get ride of the inclusion because you will access to your cache method through : `request.cherry.pluginConfigurator.getPluginInstance('identifier-of-the-plugin')`

For now, I know, you are thinking : "Well, the benefits are very low". And you're right. But then you can combine it with the hooks. We didn't see how it works, we will see it later, but just trust me. Imagine methods you can register and called at some point suring the process of the request. So yeah, you can (again) use the helper and include it in each hooks, but with the plugin you stay in a cherry context without dealing with the path for example.

Because when you retrieve the plugin, you get an instance of your class plugin, you can store this object in the request in a first hook and get it later in another hook (or even in your route callback). And an object can have datas with its attributes, so you will not have conflicts between 2 requests.

A last benefit of the plugins are the interface it should follow. For exemple, our cache plugin above can be connected to a redis. You can make another cache plugin which use the file system. Another one with something else. All these plugins can share a same interface and the same identifier, and if you want to use another cache system, you just have to change your plugin. The rest of your code will not change.

So yeah, it's all about confort and consistency. You can do it by your own, or with alternatives. It's up to you. We just provide the feature :)

We talked a little about hooks, it's time to take a look at them !
