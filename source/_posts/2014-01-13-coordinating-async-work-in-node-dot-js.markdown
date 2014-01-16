---
layout: post
title: "Coordinating async work in Node.js"
date: 2014-01-13 10:39
comments: true
categories: node.js
---

When you first move to Node.js, you need to get used to write asynchronous single thread code that does not block. For some scenarios, writing such kind of code is not trivial, specially when you have to coordinate the execution of multiple async calls in parallel and return a result after all these tasks have been completed. One thing you don't want to do in that scenario is block the main thread to wait for all the calls, which is how you normally do in other platforms. Let's discuss a very trivial example that coordinates two calls for getting information from two different sources, which is subsequently returned as a response message in express.

```javascript
exports.index = function(req, res){

	var returnResponse = false;
	var data = {};

	getName(function(err, name) { 
		if(err) return callback(err);

		data.name = name;

		if(returnResponse) 
			res.render('index', { title: 'Express', data: data });

		returnResponse = true;
    });

	getPhone(function(err, phone) { 
		if(err) return callback(err);

		data.phone = phone;

		if(returnResponse) 
			res.render('index', { title: 'Express', data: data });

		returnResponse = true;
    });
};
```

The code above calls two functions to retrieve a name and a phone from different places asynchronously. The results of the calls are temporarily stored as members of the "data" variable. The ugly part is how the "returnResponse" variable is also used to notify that each function has done its part and a response can be returned. This contains duplicated code, it's error prone and it can easily get more complicated as the number of async calls increase.   

As interesting fact, there isn't any built-in or native functionality to handle an scenario like this in Node.js. That's where an module like "async" comes in place. "Async" is a swiss knife for async work in Node.js. It provides a ton of functions for coordinating asynchronous tasks such as the ones shown in next examples. 

One of the functions you will find useful for an scenario like this is parallel. The parallel function receives an array of async functions to call, and invokes a final callback when all the functions have completed their part. 

```javascript
exports.index = function(req, res){

	async.parallel([
    	function(callback){ getName(function(err, name) { 
    		if(err) return callback(err);

    		callback(null, name); 
    	})},
    	function(callback){ getPhone(function(err, phone) {
    		if(err) return callback(err);

    		callback(null, phone); 
    	})}
	], function(err, results) {
		
		var data = {
			name : results[0],
			phone: results[1]
		}

		res.render('index', { title: 'Express', data: data });
	});
}
```

Every function in the Array passed to the parallel function receives a callback argument, which must be called after the work is completed. That callback receives two arguments, an error if something went wrong and a result. All the collected results and  errors are later passed to the final callback.

Another useful function is "each", which is similar to "parallel", but it iterates over an array and invokes a function representing the async work for every element in that array. 

```javascript
exports.index = function(req, res) {
	async.each([1, 2, 3], function(item, callback) {
       	getDataForItem(item, function(err, data) {
       		callback(null, data);
       	});

	}, function(err, data) {

       res.render('index2', { title: 'Express', data: data });
	});
}
```

In the code above, the function is executed three times with the values "1", "2" and "3". A callback is also executed when the work is completed. The final callback is executed after all the functions invoked the callback. 

This is a good way to coordinate async work in node.js, but you also have promises, which is not widely adopted yet in the platform as a way to represent async tasks. It's implemented by a lot of modules out there, but it's not something standard yet. A promise is an object that represents an asynchronous task. Among other things, this object represents a way to manipulate the task, determine when it is completed or to chain other tasks for example. The following example illustrates how a promise looks like in the Moongose module (example included in the [Moongose docs](http://mongoosejs.com/docs/api.html)),

```javascript
promise = Meetups.find({ tags: 'javascript' }).select('_id').exec();
promise.then(function (meetups) {
  var ids = meetups.map(function (m) {
    return m._id;
  });
  return People.find({ meetups: { $in: ids }).exec();
}).then(function (people) {
  if (people.length &lt; 10000) {
    throw new Error('Too few people!!!');
  } else {
    throw new Error('Still need more people!!!');
  }
}).then(null, function (err) {
  assert.ok(err instanceof Error);
});
```

Meetups is a Moongose object for executing queries against a MongoDB collection. "exec" is a method in the returned promise to start the call, and "then" is another method to chain other promises when the execution is completed. In this example, another promise to find people is executed after all the meetups have been found.






 