---
layout: post
title: "ES6 Generators - Synchronous Looking Asynchronous Code"
date: 2014-03-01 12:10:14 +0000
comments: true
categories: JavaScript es6
---
In my opinion, the most challenging thing about programming JavaScript in the browser is having to deal with asynchronicity.  As developers, we would much rather write code like this:
{% gist 9340627 %}

Our concious mind or left brain operates in what can be thought os as an abstraction of reality.  The universe is actually an infinitesimal series of events happening simultaneously at the same time that our concious mind cannot grasp, it thinks sequentially or linearly and we process one thought at a time  .We have created our computers to reflect this and for all intensive purposes the programs that we write are at their base level, a series of sequential inputs.  

The key to writing good asynchronous code is to make it look synchronous.

The unfortunate thing is that when trying to deal with asynchronicity through the use of plain callbacks, our code has often ended up like the sample below which is often referred to as **callback hell**:
{% gist 9340677 %}

I could almost live with the code above if it weren't for error handling.  Having to pass callback and errBack handlers to each call quickly turns into a tangled mess.

It is worth stating that there is absolutely nothing wrong with callbacks and they are in fact the currency of asynchronous programming and the newer asynchronous abstractions like promises would not be possible without them.  What is wrong with the above code is that it quickly becomes tightly coupled as you pass success callback and failure errorBack callbacks to each asynch call.  

Promises are probably the most commonly used alternative to callback hell and below is a refactoring of the above using the <a href="https://github.com/tildeio/rsvp.js/" target="_blank">RSVP</a> promise library.
{% gist 9340686 %}

What is nice about the above code is that we have one catch handler on **line 19** for all of the **getJSON** calls and in the event of an error in any of these calls, the error will be propagated forward to the next available error handler.

A promise represents the eventual outcome of an asynchronous or synchronous operation.  A promise is also referred to as a **thenable**, which is really just an object or a function that defines a **then** meethod.  A promise is either fulfilled in the good case or rejected in the not so good case.  A promise will execute a success callback or an error callback on completion of the asynchronous or even synchronous work.  Promises are possibly the most popular solution to callback hell that you will find in production code today.  The solution I am going to show with es6 generators makes heavy use of promises to give the illusion of synchronicity.

So if promises are where we are then es6 generators appear to be where we are headed.  I've just spent the morning playing about with the new generators feature of es6 and I was able to refactor the previous two code samples to the code below, which is almost synchronous looking:
{% gist 9340695 %}

Before I explain what is going on here, let me briefly explain what a geneator is.  There are lots of other posts on this article but here is some code that explains how a base level simple generator works:
{% gist 9340706 %}

- **Line 1** Defines a generator function using the new *** syntax**.  A generator function is a function that returns an iterator which can be thought of as an object that moves the target object on to the next value in a sequence.  This iterator exposes a **next()** interface to aid with iteration.  A javascript iterator **yields** successive **nextResult** objects which contain the yielded **value** (line 12) and a **done** flag (line 10) to indicate whether or not all of the values have been returned.

- **Line 11** assigns a returned **nextResult** object to the local object which we can check on each iteration of the loop.  When next is called, the iterator returned from the **oneToThree** function points to the next **yield** statement (**lines 2- 5**) and returns the value.  If there are no more yield statements then the **done** flag is set to true.

It is also possible to send values back to the iterator by calling **next** with an argument:
{% gist 9340726 %}

In the above example, the return of the **yield** statement is passed back to the iterator by calling **next** with the last value that was returned from the iterator and this value is then assigned to the variable on the left hand side of the **yield** statement.  Using this technique, the variables **users**, **contacts** and **companies** on lines 17, 18 and 19 can all be assigned values by passing arguments via **next**.

If you have grasped this two way communication between the calling code and iterator then you can see that the code above code is not a million miles away from the code below where **self.users**, **self.contacts** and **self.companies** on lines 6, 7 and 8 below are seemingly assigned to the results of asynchronous calls. 
{% gist 9340732 %}

The key to this magic is the **async** function call on **line 4** of the above code and below is the listing of this **async** function.
{% gist 9340745 %}

I lifted the **async** method from the <a href="https://github.com/kriskowal/q/blob/v1/q.js#L1167" target="_blank">Q promise library</a> with some minor changes to get it to integrate with promises defined in <a href="https://github.com/tildeio/rsvp.js/" target="_blank">RSVP</a> that I use on a day to day basis with ember.

Generators work syncnronously which is different than what I originally thought, I thought there was some dark witchcraft that made the calls work asynchronously.  This is not true, the key to making them work asynchronously is to return a promise or another object that describes an async task.  In this example, each **yield** statement returns a promise like this:
{% gist 9340765 %}

The basic premise that makes this possible, is that we are passing the generator function to **async** like this:
{% gist 9340788 %}

The **async** function assigns the iterator that is returned from this generator function to a local variable on **line 16**:
{% gist 9340793 %}

The async function then uses the often overlooked ability to pass arguments via the **bind** function:
{% gist 9340817 %}

The **callback** and **errback** function pointers both reference the **continuer** function on **line 2** but because of the way they were declared with the **bind** function, either **next** or **throw** will be bound to the **verb** parameter depending on which function pointer is invoked.   

When the **callback** reference is invoked on **line 20** with the **return callback();** statement, the **verb** argument will have a value of **next** which will result in the generator asking for the first value from the iterator or generator function.

{% gist 9340826 %}

Which is the equivelant of:

{% gist 9340830 %}

The **arg** argument will be **undefined** at this stage because no value has been returned from the iterator. When the first value is asked of the iterator with the above code, the code below will be executed.

{% gist 9340831 %}

The **getJSON** method returns a promise that is fulfilled or rejected with respect to the successful completion or rejection of the aynchronous ajax call.  Once the promise has been created, the **yield** statement will suspend execution of this function and return the promise created via the **getJSON** function back to the calling code or **async** function.

The **result** variable has been assigned a **nextResult** object that was described earler that will have a **done** flag value of false and a **value** property that will point to the promise created in the iterator.

The code will now encounter this **if** statment. 
{% gist 9340838 %}

As not all of the **yield** statements have been executed, the **done** flag will be false and execution will continue after the **else** statement. The next line calls **RSVP.Promise.resolve** and passes **result.value** as an argument which at this stage is still the promise returned from **getJSON**.  **RSVP.Promise.resolve** in this instance just checks that **result.value** is a promise and if not, it creates a promise that will become resolved with the passed value.  In this instance, **result.value** is a promise so no **Promise** cast is required.  

This is where things get complicated so keep your wits about you.  This **result.value** promise which was created from the **getJSON('/users')** call will resolve successfully and result in the **callback** being executed in the **promises'** **then** method below:
{% gist 9340857 %}


The **callback** will call the **continuer** function again:
{% gist 9340861 %}
The callback will call the **continuer** and bind **next** to the **verb** paramter but this time, the **arg** value will be bound to whatever was returned from the **getJSON('/users')** asynchronous call.  This means that whenever **next** is called:
{% gist 9340864 %}

The **arg** value will be returned to the iterator function as was explained in the two way communication section earlier and be assigned to the **self.users** property that was on the left hand side of the **yield** statement that called **getJSON('/users')**.

This means that **self.users** below will now be assigned the result of the async ajax call albeit in a rather round about fashion.
{% gist 9340874 %}

The **async** function then continues in this recursive manner until all of the yield statements have been called and all of the properties on the left hand side of the **yield** statements below have been assigned:
{% gist 9340878 %}

Once all the yield statements have been executed, the last **nextResult** returned from the iterator will point to **line 5** of the above code sample and will have the **done** flag set to **true**. This means that execution will continue in the **if** path of the code below and the value of the last **nextResult** object will be returned to the calling code and the process has completed:
{% gist 9340884 %}

Writing this blog post has completelly demystified what initially seemed like black magic to me.  I hope it has done the same for you.
