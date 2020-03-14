---
id: orm
title: ORM Plugins
---

The ORMs (Object relational mapping) is now a standard in frameworks, it allows to use class instances which represent your entities instead of having raw objects and it makes SQL request easier.

The ORM plugin of Cherry is kind of special. It is a core plugin like the view rendering plugin of the previous chapter and its identifier is `DatabaseEngine`. But because of its special behaviour with the connection to the database, it also has a specific manager. What does it change ? Not so much, but it provides additionnal features.

We will see later how it works, but keep in mind that the ORM plugin triggers hooks. It's the following :
- `beforeStartOrm` : Triggered right before the connection to the database
- `afterStartOrm` : Triggered right after the connection to the database
- `beforeStopOrm` : Triggered right before deconnecting the database (when Cherry is shutdown)
- `afterStopOrm` : Triggered right after deconnecting the database (when Cherry is shutdown)

In the four cases, an object with the cherry instance and the ORM plugin instance is sent as parameter.

The Cherry team provides 2 ORMs plugins for the 2 most used ORMs used in Javascript :
- [TypeORM](https://github.com/Lund-Org/cherry-typeorm-connector)
- [Sequelize](https://github.com/Lund-Org/cherry-sequelize-connector)

This plugin type is a core plugin so it needs to follow a specific API if you want to create your own connector between Cherry and an existing ORM (or if you want to code one) :

```javascript
class MyCustomORMPlugin {
  constructor () {
    /*
      The constructor of your Plugin
      Because this plugin is specific, it is not initialized with the configuration directly
    */
  }

  static getIdentifier () {
    return 'DatabaseEngine'
  }

  checkOptions (options) {
    // This method is to check if some mandatory informations are present in the database configuration
    // This options parameter is equal to the key "database" in the configuration file of Cherry
    // Save the datas you need for the connection because it's the only method which get the configuration
  }

  async connectDatabase () {
    // Manage the connection to the database thanks to the data sent in checkOptions before
    // You can manage multiple database connection if you want, you manage in this plugin the way you query
  }

  async postConnectionProcess () {
    // This method is used to add post process right after the connection
    // It could be replace with the hook after the connection but here you are in your plugin class so you can access to your instance attributes
    // For example, this method is not useful for TypeORM, but for Sequelize it's very important to register the models and tables because you can't do it before connecting the database
  }

  getConnection () {
    // Get the ORM object to query your database
  }

  async closeConnection () {
    // The method to disconnect your plugin from the database
  }
}
```

So now, you registered your plugin, it is configured, it is connected to your database, what's next ? How to use it ?

Well, it depends on your ORM of course, but you can retrieve the object which represents your database connection. With it, you can do your queries. In the request (in your route callback), you can find the cherry object. You can access to the ORM manager and call the getConnection method (which call the method with the same name from the plugin). In another words, you can code it like that :

```javascript
function yourRouteCallback (request, response) {
  const myDBConnection = request.cherry.ormManager.getConnection()
}
```

> In the future, an improvement will give more getter (or helper functions) to the cherry class to avoid browsing the manager like that

Another trick but very dependent on the implementation of the ORM, with TypeORM for example, is to retrieve the connection through the singleton provided by the library. For instance, you can do the same thing as above with this code :

```javascript
const typeorm = require('typeorm')

function yourRouteCallback (request, response) {
  const myDBConnection = typeorm.getConnection()
}
```

But we don't recommand it, the plugin has a role of abstraction and if you change for an other ORM, you will be forced to change more code (because there is not always a singleton here for you depending the ORM choosen).

Once you have the connection object, you can use your ORM like you always do.

Now, let's talk about another core plugin that can be important for you.
