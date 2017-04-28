---
layout: post
title: "Share private Node.js modules across applications (locally)"
date: 2015-11-12 14:53:11 +0100
comments: true
category: blog
tags: [Javascript, Node.js]
---

In this post I'm going to talk about a quite useful topic around Node.js: **Code Reuse**. I didn't find a good article enumerating the different options you have and the benefits and disadvantages of each of them. I have to deal with this in most of my Node projects so describing this can be of great help.
<!-- more -->
Suppose you have a bunch of private libraries you want to share on one or several applications. Let's say you developed some common functionality that can be used in different applications. You have to be able to share them across your Node applications in a centralized way, but privately, because you don't want them to be publicly accessed. These are only for our personal use.

I'm going to focus only in sharing libraries locally (not through a remote repository or whatever), which is good for development purposes. I might cover how to share libraries remotely in subsequent articles.

Below are the different options available.

* Local installation (npm install <folder>)
* Local paths (npm install --save)
* Link (npm link)
* Create your own local registry

### Local installation (npm install <folder>)
You can achieve this by using **npm install** command. Let's put an example. Suppose you have this folder structure.

```
common_lib
├──lib
|   ├──lib1.js
|   ├──lib2.js
├──package.json
├──index.js

app1
├──app.js
├──package.json
```

You want to use **common_lib** module with its submodules in **app1** application.

**lib1.js**

```js
'use strict'

function Lib1(){
	console.log("Constructing Lib1...");
}

Lib1.prototype.helloLib = function(){
	console.log("Hello Lib1");
};

Lib1.prototype.goodbyeLib = function(){
	console.log("Goodbye Lib1");
};

module.exports = Lib1;
```

**lib2.js**
```js
'use strict'

function Lib2(){
	console.log("Constructing Lib2...");
}

Lib1.prototype.helloLib = function(){
	console.log("Hello Lib2");
};

Lib1.prototype.goodbyeLib = function(){
	console.log("Goodbye Lib2");
};

module.exports = Lib1;
```

**index.js**

```js
'use strict'

var lib1 = require('./lib/lib1.js');
var lib2 = require('./lib/lib2.js');

module.exports.lib1 = lib1;
module.exports.lib2 = lib2;
```

In **package.json** from common_lib module you'll need to indicate that the module is private (to avoid accidental publishing to npm) and the main file for your module.

```json
...
"name"    : "common-lib",
"private" : "true",
"main"    : "index.js",
...
```

Then, you have to install the common lib module under your app1 application.

```sh
$ cd /projects/app1
$ npm install ../common_lib
```

The above commands created **node_modules** folder and installed common_lib module in it. Now, it's time to make use of the common libraries in my application. How would you do that?

**app1.js**

```js
'use strict'

var common_lib = require('common-lib');
var lib1 = new common_lib.lib1();
var lib2 = new common_lib.lib2();

lib1.helloLib();
lib2.helloLib();
```

This is the result of running app1.

```sh
$ node app1/app.js
Constructing Lib1...
Constructing Lib2...
Hello Lib1
Hello Lib2
```

### Local paths (npm install --save)

There's another way of installing a module locally without pointing to any remote path.	As of version 2.0.0 of Node.js you can install a local module directly using npm install --save. [This link](https://docs.npmjs.com/files/package.json#local-paths) explains well how to achieve it.

To make it accessible from app1, we would do something like this:

```sh
$ cd app1
$ npm install --save ../common-lib/
```

The above command normalized our app1/package.json and also created **node_modules/common-lib** folder. It **automatically** added the dependency to the package.json file.

```json
"dependencies": {
    "common-lib": "0.0.1"
}
```

Now it's possible to run app1 successfully as indicated in the previous example.

### Link (npm link)
The previous options are good but every time you make changes in common_lib module you have to reinstall the module in all applications which use it. If common_lib is not yet stable or you keep on adding new features to it, it can be a hassle to rebuild each time.

There's a better choice to deal with this: [**npm link**](https://docs.npmjs.com/cli/link). This is only recommended for **development environments**.

Suppose we have the same project structure as in the previous example. This is how you would achieve sharing common-lib in other projects.

```sh
$ cd app1
$ npm link ../common_lib
```

Now we have a **node_modules** folder under app1 and a link to common_lib folder inside. Every time we make a change on our common_lib the changes will be automatically reflected in our app1 application, so this option is the one I like most for development.

### Create your own local registry

Even though this option is recommendable for production environments I'm going to include it in this chapter because it could be also good in development. Creating a node repository locally is perfectly valid but this option is the most time-consuming. This involves replicating CouchDB database, implement the API's,...

I've never tried it, but if you're a Java programmer or you have some Java knowledge, the concept would be similar to installing a Maven repository such as Nexus.

This option would be recommendable when you have several modules you need to share across your organization, so installing a central private npm would be a good choice.

[Here, it's the official documentation](https://docs.npmjs.com/misc/registry).
