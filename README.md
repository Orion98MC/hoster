# A hosting rack

Hoster is an express services rack for node environment. It allows you to add/remove services to a server very easily and thus reuse your services throughout different projects.

## Requirement

Hoster requires express node packet to be installed as global (ex: npm i -g express)


## Install

```sh
$ npm install -g hoster
```

Be sure to have /usr/local/bin in your PATH


## Usage

```sh
$ hoster  [<mnt-point>:]<service.js> [[<mnt-point>:]<service.js> ...]
```  

* At least one <service.js> must be provided
* each <service.js> must conform to the hoster protocol
    
Example:

```sh
$ hoster ./db-config.js :../stats/stats.js ./server.js /blog:../blog/blog.js

```

default port is 3000, but you can change it by prepending PORT=<port-number> environment variable:

```sh
$ PORT=8080 hoster ./db-config.js :../stats/stats.js ./server.js /blog:../blog/blog.js

```

hoster defines no routes and doesn't attach any express/connect middleware.


## Hoster protocol

Hoster requires that services export a single function taking an express app as argument:

```js
module.exports = function (app) {

  /*
    configure the app...
  */

};
```

When a service has no mount point in the hoster command line, the current app is passed as argument. This kind of service can add routes, or do anything to the global express app. 

When a service requires to be mounted at a specific mount-point, a new express app is created and passed as argument. This kind of service is decoupled from the hosting context. (i.e. app.use(<mount-point>, service)) 
If the mount-point is empty (ex: hoster :./service.js) then the service is a added as a middleware with no path (i.e. app.use(service))

Services inherit their parent's 'root' (cwd) and 'views' settings accessible via app.get(). They can also access their mount-point via app.get('base'). 

If a service requires a particular setup when attached, it is possible to set the 'configurator' setting in the hosting context which is a function that takes the mount-point and the mounted app as argument:

```js
module.exports = function (app) {

  /*
    configure the app...
  */
  
  app.set('configurator', function (mnt, newApp) {
    if (mnt === '/blog') {
      newApp.set('views', __dirname+ '/blog/views');
      // Now the blog app can find it's own views...
    }
  });

};
```

# License

Copyright (c) 2014, Thierry Passeron
MIT License
