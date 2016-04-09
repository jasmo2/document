#**NodeJs** Questions

## 1. What is the difference between frontend Javascript and **NodeJs**?

**NodeJs** is a web-engine mount over *Google Chrome* **V8** Javascript's engine. This allow to run Javascript code outside a browser. Moreover, **NodeJs** provides an incredible package manager named **[NPM](https://www.npmjs.com/))** (Node Package Manager), which is for the server side; if a frontend package manager want to be use better to reference to **[Bower](http://bower.io/)**.

## 2. How many threads does **NodeJs** support?
**NodeJs** is a single thread application, which means it would only run on a single processor on your machine. There are alternatives that makes **NodeJs** use full cores capacity, by creating a cluster that share the same port, meaning use a **NodeJs** instance for each Core. The library which supports this functionality is **[Cluster](https://**NodeJs**.org/api/cluster.html)**.

## 3. How does Error handling in a synchronous way should be in **NodeJs**?
By been a single thread application errors should be handle in Node because if something happened the server could shutdown. Just look the following snipet:

```javascript
var division = function(a,b){
  if (b===0)
    throw new Error('oh no, division by zero!')

  return a/b;
}

// Main App
var result = division(8,2)
console.log(result) // result = 4

var result2 = division(8,0)
// Error: oh no, division by zero!
//     at division (/tmp/09b590d0-fb8e-11e5-93a6-eb8570ecb82e/Node Application/app.js:3:12)
//     at Object. (/tmp/09b590d0-fb8e-11e5-93a6-eb8570ecb82e/Node Application/app.js:12:15)
//     at Module._compile (module.js:449:26)console.log(result2);

```

To handle the error you could use **[domain](https://**NodeJs**.org/api/domain.html)**, but is going to be relatively soon to be deprecated. On the other hand, **NodeJs** 5.x have its own error handler *try catch finally*.  On the next code an non existing dependency is going to be handle:

```javascript
const assert = require('assert');
try {
  fakeDependency;
} catch(err) {
  assert(err.arguments[0], 'fakeDependency');
}
```
On the previous snipet the `ReferenceError` is catch by the block.

>A JavaScript exception is a value that is thrown as a result of an invalid operation or as the target of a throw statement. While it is not required that these values are instances of Error or classes which inherit from Error, all exceptions thrown by Node.js or the JavaScript runtime will be instances of Error.

## 4. How could a **NodeJs** application restart all alone?

An actual paradigm use by many enterprises is the eventually the application will crash, but in an ideal case it should restart but it self without human intervene. On NPM ecosystem there are some alternatives that will to execute this task. some of the libraries are:
+ [nodemon](https://www.npmjs.com/package/nodemon)
+ [node-supervisor](https://github.com/petruisfan/node-supervisor)
+ [forever](https://www.npmjs.com/package/forever)

To illustrate the usage of fault tolerant libraries the next example is going to be an *Express* application.
Before any code is good practice to install `nodemon`  it globally:
> terminal

```
 npm install -g nodemon
```

Then, check application's`pacakage.json` to include it inside your dependencies. Remember `scripts` tag and write `nodemon` instead of plain `npm`

> pacakage.json

```json
{
  "name": "abcquestions",
  "version": "1.0.0",
  "description": "node questions",
  "scripts": {
    "start": "nodemon ./bin/www"
  },
  "engines": {
    "node": "~5.10.0",
    "npm": "~3.8.5"
  },
  "keywords": [
    "questions",
    "abc",
    "abcquestions"
  ],
  "author": "jasmo2",
  "license": "MIT",
  "dependencies": {
    "better-console": "^0.2.4",
    "body-parser": "~1.13.2",
    "debug": "~2.2.0",
    "express": "^4.13.4",
    "jade": "~1.11.0",
    "mongoose": "^4.4.6",
    "morgan": "~1.6.1",
    "repeat": "0.0.6",
    "serve-favicon": "~2.3.0",
    "socket.io": "^1.4.5",
    "sticky-session": "^1.0.2",
    "nodemon": "^1.9.1"
  }
}

```


## 5.  How to code real time applications ?
**NodeJs** is well know for real time applications, specially for heavy *WebSockets* usage. For instance, its more popular *WebSocket* library is call *[socket.io](http://socket.io/)*. On the next snipets two types of using sockets are going to be presented:

> Zero libraries

```javascript
// Socket instance
var mySocket = new WebSocket('ws://localhost:3000');

// Open the socket
mySocket.onopen = function(event) {

	// Send an initial message
	mySocket.send('Hello WebSocket client');

	// Listen for messages
	mySocket.onmessage = function(event) {
		console.log('WebScoket message',event);
	};

	// Listen for socket closes
	mySocket.onclose = function(event) {
		console.log('WebSocket closed',event);
	};

	// To close the socket....
	socket.close()

};
```

> Using **Socket.io**

```javascript
// Create express application
var express = require('express');
var app = express();

// Start app and listen on port 3000 for connections
var server = app.listen(3000, function () {
  var host = server.address().address;
  var port = server.address().port;

  console.log('Example app listening at http://%s:%s', host, port);
});

var io = require('socket.io')(server);	//Binds socket to http server
io.on('connection', function(socket) {
  client.on('message',function(event){
		console.log('Hello WebSocket client', event);
	});
	client.on('disconnect',function(){
		clearInterval(interval);
		console.log('WebSocket closed');
	});
});
```

## 6.  What is the **"_"** variable for?
This question seems easy to developers, but the truth is that not every one know what does it mean.
> The underscore variable takes the last result that has been use an store it momentaneously.

```javascript
var a = 2;
var b = 6;

var c = a + b
c;
var sum = _

console.log("var c: "+c+" ;var sum: "+sum); //
// var c: 8 ;var sum: 8
```


## 7.  What does a **Callback** mean in **NodeJs**?
A callback is an equivalent to function but asynchronous, it is in charge to complete a given task. For instance, on the next example is going to be be shown how to use callbacks:
> responseHandler.js

```javaScript
var ctrEvents = require('../controllers/eventController');

ctrEvents.EventValidate(data,function(){
          sendJsonResponse(res, 201, location);
		});
```

>EventController.js

```javaScript
// ... dependencies
var self = module.exports = {
  EventValidate: function(data, callback){
      item.findOne({idItem: data.idItem, idSubItem: data.idSubItem}).stream()
          .on('data', function(item){
              if (data.Item === item.SubItem){
                  console.log('Super item');
                  //self.sendMessageTwilio(data,'Su mascota ha muerto de un paro cardiaco');
              }
              else {
                  console.log('no super Item');
              }
          })
          .on('error', function(err){
              // handle error
            console.log('error validate');
          })
          .on('end', function(){
              // final callback
            callback();
          });
  },
}
```


## 8.  Explain the meaning of **Callback hell** ?
On **NodeJs** Callbacks are a rule coding. But, heavy couple code, which is messy to read or modify.

On the following example we are supposing we are going to use `mongoose` as a ORM:

```javascript
Blog.findById(req.body.BlogId, function(err, blog){
      if(err) return next(err);

      Comment.findById(req.body.commentId, function(err, comment){
        if(err) return next(err);

        Tag.findById(req.body.tagId, function(err, tag){
          if(err) return next(err);

          if(tag.isAvailable()){
            tag.useIn.push(blog.id);
            tag.save(function(err){
              if(err) return next(err);
              return res.json({message : 'Tag created successful'});
            });
          } else {
            //Call error handler
            return next({message : 'Tag already taken'});
          }
        })
      });
    });
```

Is seen that there are 3 levels on the calls before is verify is *tag is available*. A better way is to use promises if the third party library supporst them.

```javascript

Blog.findById(req.body.BlogId).exec()

    //Get the Blog
    .then(function(commentFromDb){
      return Comment.findById(req.body.commentId).exec();
    })
    //get the Comment and load the Tag
    .then(function(tagFromDb){
      tag = tagFromDb
      return Tag.findById(req.body.tagId).exec();
    })
    //Check the tag availability
    .then(function(tag){
      if(tag.isAvailable()){
        tag.useIn.push(blog.id);
        return tag
    })
    //Save the save the Tag
    .then(function(tag){
      tag.save
    })
    //Answer the petition back
    .then(function(tag){
      return res.json({message : 'Enrollment successful'})
    });
    //Catch all errors and call the error handler;
    .then(null, next);
```

Now the code is cleaner and more readable.

## 9. How should the callback convention be?

The errors should be first when modules expose their methods. For instance:

```javascript
module.exports = function (conventionVar, callback) {  
  var stuff = doStuff(conventionVar, function(err, data) {
      // here we check, if an error happened
      if (err) {
        // an error has raise and the callback pass it
        callback(err);
      }
    // note, that the first parameter is the error
    // which is null here
    // but if an error occurs, then a new Error
    // should be passed here
  })
  return callback(null, stuff);
}
```


## 10. Configuration file **.js** or **.json** ?
In the Javascript universe is very common to use configuration files on *.json* files. But, *.js* files provides more flexibility.
For instance:

> config/db.js

```javascript

var mongoose = require('mongoose');

var dbUri = 'mongodb://mymongoBD';
if(process.env.NODE_ENV === 'production'){
    dbUri = process.env.MONGOLAB_URI;
}

//connect
mongoose.connect(dbUri);

//mongoose events
mongoose.connection.on('connected', function(){
    console.log('Mongoose connected to: ' + dbUri);
});
mongoose.connection.on('error', function(err){
    console.log('connection error: ' + err);
});
mongoose.connection.on('disconnected', function(){
    console.log('Mongoose disconnected');
});

var gracefullShutdown = function(msg, callback){
  mongoose.connection.close(function(){
      console.log('Mongoose disconnected through ' +  msg);
      callback();
  });
};

//Shutdown events
//Nodemon shutdown
process.once('SIGUSR2', function(){
    gracefullShutdown('nodemon restart', function(){
       process.kill(process.pid, 'SIGUSR2');
    });
});

//App termination
process.once('SIGINT', function(){
    gracefullShutdown('app termination', function(){
        process.exit(0);
    });
});

//Heroku shutdown
process.once('SIGTERM', function(){
    gracefullShutdown('Heroku app hutdown', function(){
        process.exit(0);
    });
});
```

> main.js

```javascript


require ('./config/db.js')
var mongoose = require('mongoose');

var userSchema = new mongoose.Schema({
	idUsuario: {type: String, required: true},
	phone: {type: String, required: true}
});

mongoose.model('user', userSchema);

...
```

On the previous snipets is shown how a *config.js* is useful, helping encapsulating a concern and initializing the database connection.
