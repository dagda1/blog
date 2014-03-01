---
layout: post
title: "ES6 Generators - Synchronous Looking Asynchronous Code"
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


The left part of our brain brains think sequentially and these thoughts are really an abstraction of reality.  The universe is actually an infinitesimal series of events happening simultaneously at the same time but we think sequentially, we process one thought at a time and we have created our computers to reflect this and for all intensive purposes programs are a series of sequential inputs.  

I think it is safe to say that the real goal of writing asynchronous code is to make it look synchronous.

The unfortunate thing is that when trying to deal with asynchronicity with the use of callbacks, our code has often ended up like below which is often referred to as **callback hell**:
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

It is worth noting that there is absolutely nothing wrong with callbacks and they are in fact the currency of asynchronous programming and the newer asynchronous abstractions like promises still make heavy use of them and indeed would be impossible without them.  What is wrong with the above code is that it quickly becomes tightly couple as you pass success callback and failure errorBack callbacks to each asynch call.  

Promises are probably the most commonly used alternative to callback hell and below is a refactoring of the above:
{% codeblock %}
// FINISH THIS
{% endcodeblock %}

A promise represents the eventual outcome of an asynchronous or synchronous operation.  A promise is also referred to as a **thenable**, which is really just an object or a function that defines a **then** meethod.  A promise is either fulfilled in the good case or rejected in the not so good case.  A promise will execute a success callback or an error callback on completion of the asynchronous or even synchronous work.  Promises are possibly the most popular alternative to the humble callback that you will find in production code.  The solution I am going to show with es6 generators still makes use of promises to achieve the asynchronous results.

So if promises are where we are then es6 generators appears to be where we are headed.  I've just spent the morning playing about with the new generators feature of es6 and I was able to come up with the code sample below which looks a whole lot more synchronous:
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

It is also possible to send values back to the iterator by calling next with an argument:
{% codeblock %}
function BulkLoader(){
  this.users = [];
  this.companies = [];
  this.contacts = [];
};

BulkLoader.prototype.iterator = function * () {
  // var a = yield 1;
  // var b = yield 2;
  // var c = yield 3;

  this.users = yield [{name: 'bob', email: 'bob@hotmail.com'}];
  this.contacts = yield [{name: 'brian', email: 'brian@hotmail.com'}];
  this.companies = yield [{name: 'company', email: 'company@hotmail.com'}];

  return this;
};

var users = iterator.next();
var contacts = iterator.next(users.value);
var companies = iterator.next(contacts.value);

var result = iterator.next(companies.value).value;

console.log(result.users);
console.log(result.contacts);
console.log(result.companies);
{% endcodeblock %}

In the above example, the return of the **yield** statement is passed back to the iterator where it is assigned to the variable on the left hand side of the **yield** statement.  Using this technique, the variables **users**, **contacts** and **companies** on lines 12, 13 and 14 can all be assigned values by passing arguments via **next**.

If you have grasped this two way communication between the calling code and iterator then you can see that the code above code is not a million miles away from the code below where **self.users**, **self.contacts** and **self.companies** on lines 6, 7 and 8 below are seemingly assigned to the results of asynchronous calls. 
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
Generators work syncnronously which is different than what I originally thought, I thought there was some magic that made them work asynchronously.  This is not true, the key to making them work asynchronously is to return a promise or another object that describes an async task.  In this example, each **yield** statement returns a promise like this:
{% codeblock %}
self.users = yield getJSON('/users');
{% endcodeblock %}

I got the **async** method from the <a href="https://github.com/kriskowal/q/blob/v1/q.js#L1167" target="_blank">Q promise library</a> with some minor changes to get it to integrate with promises defined in <a href="https://github.com/tildeio/rsvp.js/" target="_blank">RSVP</a> that I use on a day to day basis with ember.

There is a lot going on in the 2 code sniippets above but the basic premise is that we are passing the generator function to **async** like this:
{% codeblock %}
  return async(function * () {
{% endcodeblock %}
The **async** function assigns the iterator that is returned from this generator function to a local variable on **line 16**:
{% codeblock %}
  var generator = generatorFunc();
{% endcodeblock %}
The async function then uses the often overlooked ability to pass arguments via the **bind** function:
{% codeblock %}
var callback = continuer.bind(continuer, "next");
var errback = continuer.bind(continuer, "throw");
{% endcodeblock %}
The **callback** and **errback** function pointers both reference the **continuer** function on **line 2** but because of the way they were declared with the **bind** function, either **next** or **throw** will be bound to the **verb** parameter depending on which function pointer is invoked.   

When the **callback** reference is invoked on **line 20** with the **return callback();** statement, the **verb** argument will have a value of **next** which will result in the generator asking for the first value from the iterator or generator function.

{% codeblock %}
result = generator[verb](arg);
{% endcodeblock %}

Which is the equivelant of:

{% codeblock %}
result = generator.next(arg);
{% endcodeblock %}

The **arg** argument will be undefined at thes stage because we have not had a value returned from the iterator. When the first value is asked of the iterator with the above code, the code below will be executed.

{% codeblock %}
self.users = yield getJSON('/users');
{% endcodeblock %}
The **getJSON** method returns a promise that is fulfilled or rejected with respect to the successful completion or rejection of the aynchronous ajax call.  Once the promise has been created, the **yield** statement will suspend execution of this function and return the promise created via the **getJSON** function back to the calling code or **async** function.

The **result** variable has been assigned a **nextResult** object that was described earler that will have a **done** flag value of false and a **value** property that will point to the promise created in the iterator.

The code will now encounter this if statment. 
{% codeblock %}
if (result.done) {
  return result.value;
} else {
  return RSVP.Promise.resolve(result.value).then(callback, errback);
}
{% endcodeblock %}
As not all of the **yield** statements have been executed, the **done** flag will be false and execution will continue after the **else** statement. The next line calls **RSVP.Promise.resolve** and passes **result.value** as an argument which at this stage is still the promise returned from **getJSON**.  **RSVP.Promise.resolve** in this instance just checks that **result.value** is a promise and if not, it creates a promise that will become resolved with the passed value.  In this instance, **result.value** is a promise so no **Promise** cast is required.  

This is where things get complicated so keep your wits about you.  This **result.value** promise which was created from the **getJSON('/users')** call and will resolve successfully and result in the **callback** being executed in the **promises'** **then** method below:
{% codeblock %}
return RSVP.Promise.resolve(result.value).then(callback, errback);
{% endcodeblock %}

The **callback** will call the **continuer** function again:
{% codeblock %}
function continuer(verb, arg) {
{% endcodeblock %}

The callback will call the **continuer** and bind **next** to the **verb** paramter but this time, the **arg** value will be bound to whatever was returned from the **getJSON('/users')** asynchronous call.  This means that whenever **next** is called:
{% codeblock %}
result = generator[verb](arg);
{% endcodeblock %}

The **arg** value will be returned to the iterator function as was explained in the two way communication section earlier and assigned to the **self.users** property that was on the left hand side of the **yield** statement that called **getJSON('/users')**.

This means that **self.users** below will now be assigned the result of the async ajax call albeit in a rather round about fashion.
{% codeblock %}
  self.users = yield getJSON('/users');
{% endcodeblock %}

The **async** function then continues in this recursive manner until all of the yield statements have been called and all of the properties on the left hand side of the **yield** statements below have been assigned:
{% codeblock %}
self.users = yield getJSON('/users');
self.contacts = yield getJSON('/contacts');
self.companies = yield getJSON('/companies');

return self;
{% endcodeblock %}

Once all the yield statements have been executed, the last **nextResult** returned from the iterator will point to **line 5** of the above code sample and will have the **done** flag set to **true**. This means that execution will continue in the **if** path of the code below and the value of the last **nextResult** object will be returned to the calling code and the process has completed:
{% codeblock %}
if (result.done) {
  return result.value;
} else {
  return RSVP.Promise.resolve(result.value).then(callback, errback);
}
{% endcodeblock %}

Writing this blog post has completelly demystified what initially seemed like black magic to me.  I hope it has done the same for you.
