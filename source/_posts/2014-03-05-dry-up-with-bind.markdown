---
layout: post
title: "Partial Functions with bind"
date: 2014-03-05 09:33:52 +0000
comments: true
categories: JavaScript
---
I gave a talk last night at the local <a href="" target="_blank">Glasgow JavaScript user's group</a> and I was a little surprised that not everybody had heard of the **bind** method which is a member of the **Function.prototype**.  I put the blame for this firmly at internet explorer's door because most of us still have to deal with intenet explorer 8 and below which runs **ECSMAScript 3** and **bind** is only available from **ECSMAScript 5** and beyond.  Internet Explorer has a nuclear half life that is much longer than Chernobyl and will probably still be emitting deadly radiation for many years to come.

The **bind** method is one solution to having to declare a **self** or a **that** or a **_this** variable when we want to specify what **this** will be when a function is executed.  Below is an example where two new functions (**lines 12 and 16**) are created in order to set what the **context** will be when the two variations of the  **sayHello** methods are executed for a particular person:
{% gist 9364974 %}

We can tidy this up by using **Function.prototype.bind**:
{%gist 9365057 %}

**Lines 12 and 13** use **bind** to create a new function that when called, has its **this** value set to whatever the given value between the parenthisis is.

It is worth mentioning that if you want to use **bind** in older browsers then you still can by using a polyfill such as <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#Compatibility" target="_blank">this</a> and I would emplore everybody who is reading this to do just that if you are not already doing so.

###Partial Functions

What is less known about bind is that we can use it to make a function with pre-specified initial arguments.  These arguments if supplied, follow the provided **this** value and are inserted at the start of the arguments passed to the target function.

Let me illustrate this with an example, the code below contains a chain of promises with almost identical resolve handlers.  I know I could use **RSVP.hash** for this but I am merely illustrating the point:

{% gist 9365442 %}

On **lines 4, 8 and 9** of the above gist, I am setting a property of the **self** reference to the data returned from the async call and on lines **6, 10 and 14** I am calling **getJSON** with a different url that will return the next resource.  The only things that are different are the property name and the string literal for the next url.

The function below would DRY this up nicely:
{% gist 9365578 %}

The only problem I have is that the above function's argument list does not fit into my resolve handlers argument list where only one argument is passed containing the result of the async call.  The answer is of course to use **bind** and specify some pre-specified initial arguments:
{% gist 9365549 %}

On **lines 9 - 11**, I am supplying some pre-specified arguments to the **bind** function that will be bound to the first two arguments of the **continuer** function when the specific instances are called.  I think this DRYs up things quite nicely.