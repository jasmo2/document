#NodeJS Questions

## 1. What is the difference between frontend Javascript and NodeJs?

NodeJS is a web-engine mount over *Google Chrome* **V8** Javascript's engine. This allow to run Javascript code outside a browser. Moreover, **NodeJs** provides an incredible package manager named **[NPM](https://www.npmjs.com/))** (Node Package Manager), which is for the server side; if a frontend package manager want to be use better to reference to **[Bower](http://bower.io/)**.

## 2. How many threads does **NodeJs** support?
**NodeJs** is a single thread application, which means it would only run on a single processor on your machine. There are alternatives that makes NodeJs use full cores capacity, by creating a cluster that share the same port, meaning use a **NodeJs** instance for each Core. The library which supports this functionality is **[Cluster](https://nodejs.org/api/cluster.html)**.

## 3. How does Error handling should be done in **NodeJs**?
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

To handle the error you could use **[domain](https://nodejs.org/api/domain.html)**, but is going to be relatively soon to be deprecated. On the other hand, NodeJs 5.x have its own error handler *try catch finally*.  On the next code an non existing dependency is going to be handle:

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

## 4.  What does a **Callback** mean in NodeJs?
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


## .  Explain “Callback hell”?
