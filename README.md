# A hosting rack

Hoster is an *ExpressJS* services rack for node environment. It allows you to add/remove services to a server very easily and thus reuse your services throughout different projects.

## Requirement

Hoster requires *ExpressJS* node package to be installed as global

Example:

```
npm i -g express
```


## Install

Hoster should be installed as global:

```sh
$ npm install -g express_hoster
```

Be sure to have /usr/local/bin in your PATH


## Usage

```sh
$ hoster  [[+]<mnt-point>:]<service.js> [[[+]<mnt-point>:]<service.js> ...]
```  

* At least one service must be provided
* each service must conform to the hoster protocol (see below)
    
Example:

```sh
$ hoster ./db-config.js :../stats/stats.js ./server.js /blog:../blog/blog.js

```

default port is 3000, but you can change it by prepending PORT=port-number environment variable:

```sh
$ PORT=8080 hoster ./db-config.js :../stats/stats.js ./server.js /blog:../blog/blog.js

```

hoster defines no routes and doesn't attach any express/connect middleware.

## Service

A service is just a javascript file that exports a handler that conforms to the hoster protocol.

Services are loaded in the command line order.

Based on how you instruct hoster to load them (in the command line), different results can be obtained.


## Hoster protocol

Hoster creates one express app and attach services to it.

Hoster requires that services export a single function taking an express app as first argument:

```js
module.exports = function myService1(app) {

  /*
    configure the app...
  */

};
```

* No mount-point services

When a service has no mount point in the hoster command line (no colon in front of its path), the current app is passed as argument. This kind of service can add routes, or do anything to the global express app automatically created by hoster. 

* Mount-point services

When a service requires to be mounted at a specific mount-point, a new express app is created and passed as argument. This kind of service is decoupled from the hosting context. (i.e. app.use(mount-point, service)) 

Features are special case of mount-point services, their mount point must start with a + (plus) sign (see features below)

* Empty mount-point

If the mount-point is empty (ex: hoster :./service.js) then the service is a added as a middleware with no path (i.e. app.use(service))


## Settings

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

## Features

Features are like services but services that are meant to be loaded at a specific point in your "regular" services.

So you load features juste like any other mount-point service but with a + (plus) sign in front of their mount point like this:

```sh
hoster +/login:login.js server.js
```

This will require the login.js and save it as a feature named '/login', the mount point here is _only_ used as the name of the feature.

Later in any service you can include this feature with the following code:

```js
if (app.features) app.features('/login');
```

This will call the login.js handler and pass it the app as argument just like with mounted services.

You can pass parameters to the app.features() these parameters will be passed to the handler:
```js
app.features('/login', { verbose: true });
```

This should work if the handler is designed to have parameters like:
```js
module.exports = function handler(app, options) {
  ...
};
```


# License

Copyright (c), 2014 Thierry Passeron

The MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.