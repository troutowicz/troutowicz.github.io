---
layout: post
title: Promises and Node.js
---

While on Christmas vacation, I had the urge to learn something new. Already having experience with JavaScript, I ended up choosing [Node.js](http://nodejs.org). If I had to pick one thing that really stood out with Node.js, it would have to be it's asynchronous nature. I would recommend the core workshops from [nodeschool.io](http://nodeschool.io/#workshoppers) to anyone looking for a challenge based introduction to Node.js.

As a final test to learning the basics of Node.js, I decided to write a module and publish to [NPM](https://www.npmjs.com). The module needed to communicate with the API of a website.

To start, I decided I wanted to avoid using any non-core dependencies. The module's code initially looked similar to:

```js
var http = require('http'),
    url = API_ENDPOINT;

function queryApi(callback) {
  http.get(url, function (response) {
    response.setEncoding('utf8');
    var data = '';

    response.on('data', function (chunk) {
      data += chunk;
    });

    response.on('end', function () {
      if (response.statusCode !== 200) {
        return callback(new Error('HTTP request failed with code ' + response.statusCode));
      }

      var result;
      try {
        result = JSON.parse(data);
      } catch(e) {
        return callback(new SyntaxError('JSON parse failed'));
      }

      callback(null, result);
    });
  }).on('error', function(e) {
    callback(e);
  });
}

module.exports.queryApi = queryApi;
```

```js
var api = require('./module');

api.queryApi(function (err, data) {
  if (err) {
    console.error(err.message);
    return;
  }

  console.log(data);
});
```

And with that, I had a working module.

As I was preparing to publish to NPM, I came across the [Q](https://github.com/kriskowal/q) library. Q provides the means to create promises in JavaScript. These promises provide another way to handle asynchronous functions.

As I am quite new to the callback/promise scene, I won't be venturing too deep into what I perceive the pros/cons of each to be... [James Coglan](https://blog.jcoglan.com/2013/04/01/callbacks-promises-and-simplicity/) and [Drew Crawford](http://sealedabstract.com/code/broken-promises/) have that pretty well covered. But I will show what this module looks like after replacing callbacks with a promise/callback hybrid approach, and briefly discuss what I think are the more significant, immediately identifiable differences between the two. If you are looking for something more, take a look at the two links above, and the articles listed at the end of this post.

Below is what the module looks like when using the [Q-IO](https://github.com/kriskowal/q-io) library (provides promisfied versions of Node's I/O functions):

```js
var http = require('q-io/http'),
    url = API_ENDPOINT;

function queryApi(callback) {
  return http.read(url)
    .then(function (body) {
      try {
        return JSON.parse(body);
      } catch (e) {
        throw new SyntaxError('JSON parse failed');
      }
    }).nodeify(callback);
}

module.exports.queryApi = queryApi;
```

```js
var api = require('./module');

var data = api.queryApi();
data.then(function (data) {
  console.log(data);
})
.catch(function (err) {
  console.error(err.message);
});
```

Comparing the two module revisions should reveal a couple differences.

In the initial version, a callback function is passed to `queryApi()`. When the method has finished its task or encountered an error, the callback is fired. The callback then performs whatever task it has been instructed to do on the result of `queryApi()`.

For the promisfied version, the eventual result of `queryApi()` is immediately returned. That eventual result is called a promise. The result of a promise is considered time independent. This means that instead of waiting for a callback to fire, we are given a placeholder for the result that can be consumed. The `then()` method is responsible for unwrapping the result found inside the promise container. The method accepts two optional function parameters (known as handlers), one for fulfillment and one for error. Once `then()` obtains the promise's result, the result is passed to the correct handler.

The most discernible difference to me is how promises provide the ability to write asynchronous code in a synchronous like fashion. Unlike callbacks, promises behave similarly to a try/catch statement. Either a value is successfully returned with `then()`, or an error is thrown and caught with `catch()`. And just like error propagation in try/catch, errors propagate with promises. When an error is thrown, it will bubble up until the first available error handler is found.

Compare how the promisfied module was used against this synchronous code:

```js
var api = require('./moduleSync')

try {
  var data = api.queryApiSync();
  console.log(data);
} catch (e) {
  console.error(e.message);
}
```

A function is fired, and if the function completes successfully, a value is returned. If the function throws an error, it bubbles up and gets caught. Promises follow the same pattern. With functions that use callbacks, however, the steps for handling success/failure must be determined in advance. Once a callback is passed to a function, the function decides where and when to fire the callback, passing to the callback the functions result.

So now that the module has been rewritten using the Q-IO library, it can now be referenced using callbacks or promises. This was a great learning exercise, but why would anyone want to use promises when they were removed from Node's core in favor of callbacks? There are very mixed feelings on this subject. It seems to come down to whether or not you agree with Node's design philosophy on how control flow should be handled.

In my case, I have not worked with Node long enough to make a constructive argument on why I think one style should be preferred over the other, if that can even be said... it seems both have strengths and weaknesses in different scenarios. But, from the time spent writing this module, I can say the biggest benefit for me was being able to take a more synchronous approach to handling an asynchronous task. Make sure to check out the linked articles to see several more differences and, depending where you sit, possible improvements to native callbacks.

[My module](https://github.com/troutowicz/node-yesnowtf)

Links to explore:

[Promise anti-patterns](http://taoofcode.net/promise-anti-patterns/)<br>
[Callbacks are imperative, promises are functional](https://blog.jcoglan.com/2013/03/30/callbacks-are-imperative-promises-are-functional-nodes-biggest-missed-opportunity/)<br>
[Promises in Node.js with Q – An Alternative to Callbacks ](http://strongloop.com/strongblog/promises-in-node-js-with-q-an-alternative-to-callbacks/)
