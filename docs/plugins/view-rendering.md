---
id: view-rendering
title: View rendering plugins
---

In this chapter, we will take a closer look at the view rendering plugin you already used.

As you already know, there is a default behaviour to render static files or html strings. This behaviour is override by the plugin you load. How does it work ?

On the load of the framework, you provide a plugin array with the class which represent your plugin. Every single plugin must have an identifier. For the view rendering plugins, this identifier is `ViewEngine`.

Because the plugins are store in a classic object, if you register multiple plugins with the same identifier, the last one will be used (an object is a simple key/value).

Because this is a core plugin, it is used by the framework, so it means the plugin should follow a specific interface. Here is what it looks like :

```javascript
class MyNewViewRenderingPlugin {
  constructor (configuration = {}) {
    /*
      The constructor of your plugin
      It takes a configuration as parameter which allows you to provide additionnal datas to customize the behavior of your plugin
      For example, the Handlebars plugin we talked in a previous chapter check if a key "viewEngine" is in the configuration and use it as parameter when it reads the html file
      The configuration is the exact same you inject in Cherry during the initialization
    */
  }

  /**
   * The method that EVERY plugins needs to have. It defines what kind of plugin it is
   */
  static getIdentifier () {
    return 'ViewEngine'
  }

  async loadTemplate (filePath) {
    // The method which will read the template file
  }

  useHtml (source) {
    // The method that compile the raw html string you can sent
  }

  bindTemplate (variables = {}) {
    // Bind variables in your html template
  }

  getRenderedHtml () {
    // The method which returns the final html
  }
}
```

By following this API, you can create a view rendering plugin for every templating engine you want. Or even code your own template language in the plugin.

> This plugin is only used with the built-in method "html" attached to the response object of a request
