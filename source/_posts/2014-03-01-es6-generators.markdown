---
layout: post
title: "es6-generators"
date: 2014-03-01 12:10:14 +0000
comments: true
categories: JavaScript es6
---
In my opinion, the most challenging thing about programming JavaScript in the browser is having to deal with asynchronicity.  As developers, we would much rather write code like this:
{% codeblock %}
try {
  var users = getJSON('/users');
  var companies = getJSON('/companies');
  var contacts = getJSON('/contacts');
 
  return [users, companies, contacts];
} cacth (e) {
  //handler error
}	
{% endcodeblock %}


The left part of our brain brains think sequentially and these thoughts are really an abstraction of reality.  The universe is actually an infinitesimal series of events happening simultaneously at the same time but we think sequentially, we process one thought at a time and we have created our computers to reflect this were for all intensive purposes programs are a series of sequential inputs.  

I think it is safe to say that the real goal of writing asynchronous code is to make it look synchronous.

The unfortunate thing is that when trying to deal with asynchronicity with the use of callbacks, our code has often ended up like below which is the artist also formerly known as **callback hell**:
{% codeblock %}
$.getJSON('/login').done(function(data) {
  $.getJSON('/users').done(function(data) {
    self.users = data;
    $.getJSON('/companies').done(function(data) {
      self.companies = data;
      $.getJSON('/contacts').done(function(data) {
        self.contacts = data;
        callback(self);
      });
    });
  });
});
{% endcodeblock %}

What a mess!  Promises are probably the most commonly used alternative to callback hell and below is a refactoring of the above:
{% codeblock %}
// FINISH THIS
{% endcodeblock %}

A promise is also referred to as a **thenable**, which is really just an object or a function that defines a **then** meethod.  A promise is either fulfilled in the good case or rejected in the not so good case.  A promise will execute a success callback or an error callback on completion of the asynchronous or even synchronous work.  Promises are a nice next step but the code is still quite removed from our orignal example of what we want to achieve.

So if promises are where we are, es6 generators appears to be where we are headed.  I've just spent the morning playing about with the new generators feature of es6 and was able to come up with the code sample below which looks a lot more synchronous:
{% codeblock %}
BulkLoader.prototype.load = function() {
  var self = this;

  return async(function * () {
    try {
      self.users = yield getJSON('/users');
      self.contacts = yield getJSON('/contacts');
      self.companies = yield getJSON('/companies');

      return self;
    } catch (err) {
      throw err;
    }
  });
};
{% endcodeblock %}

Before I explain what is going on here, let me briefly explain what a geneator is.  There are lots of other posts on this article but here is some code that explains how a base level simple generator works:
{% codeblock %}
function* oneToThree() {
  yield 1;
  yield 2;
  yield 3;
}

var iterator = oneToThree(),
    obj = {};

while(!obj.done) {
  obj = iterator.next();
  console.log(obj.value)
};
{% endcodeblock %}

- **Line 1** Defines a generator function using the new *** syntax**.  A generator function is a function that returns an iterator which can be thought of as an object that moves the target object on to the next value in a sequence.  This iterator exposes a **next()** interface to aid with iteration.  A javascript iterator **yields** successive **nextResult** objects which contain the yielded **value** (line 12) and a **done** flag (line 10) to indicate whether or not all of the values have been returned.

- **Line 11** assigns a returned **nextResult** object to the local object which we can check on each iteration of the loop.  When next is called, the iterator returned from the **oneToThree** function points to the next **yield** statement (**lines 2- 5**) and returns the value.  If there are no more yield statements then the **done** flag is set to true.

So that example is still quite far removed from how we got here:
{% codeblock %}
BulkLoader.prototype.load = function() {
  var self = this;

  return async(function * () {
    try {
      self.users = yield getJSON('/users');
      self.contacts = yield getJSON('/contacts');
      self.companies = yield getJSON('/companies');

      return self;
    } catch (err) {
      throw err;
    }
  });
};
{% endcodeblock %}
The key to this magic is the **async** function call on **line 4**:
{% codeblock %}
export default function async(generatorFunc) {
  function continuer(verb, arg) {
    var result;
    try {
      result = generator[verb](arg);
    } catch (err) {
      return RSVP.Promise.reject(err);
    }
    if (result.done) {
      return result.value;
    } else {
      return RSVP.Promise.resolve(result.value).then(callback, errback);
    }
  }

  var generator = generatorFunc();
  var callback = continuer.bind(continuer, "next");
  var errback = continuer.bind(continuer, "throw");

  return callback();
}
{% endcodeblock %}

I got this from the <a href="https://github.com/kriskowal/q/blob/v1/q.js#L1167" target="_blank">Q promise library</a> with some minor changes to get it to integrate with promises defined in <a href="https://github.com/tildeio/rsvp.js/" target="_blank">RSVP</a> that I use on a day to day basis with ember.

There is a lot going on in the above 2 code sniippets but the basic premise is that we are passing the generator function to **async** like this:
{% codeblock %}
  return async(function * () {
{% endcodeblock %}

async hands control of the function over to the scheduler, which assumes the function will yield promises and will send the values back once the promises are fulfilled

http://blog.alexmaccaw.com/how-yield-will-transform-node