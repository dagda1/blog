---
layout: post
title: "Unit Testing Ember, A.K.A. How I Learned to Love the Runloop and Stop Worrying"
date: 2012-04-03 07:55
comments: true
categories: JavaScript Ember
---
I am currently using <a href="http://emberjs.com/">Ember.js</a> for a side project and I have ran into some interesting behaviour while driving the development of the front end code test first.  I am using the excellent javascript bdd library <a href="http://pivotal.github.com/jasmine/" target="_blank">Jasmine</a> as my test runner and I must issue a warning that the code examples listed below are in coffeescript.  I have also been using the excellent <a href="https://github.com/netzpirat/guard-jasmine">guard-jasmine</a> gem to automatically run my tests when any files are modified which really is a great experience.  

I had vaguely heard of the Ember <a href="http://blog.sproutcore.com/the-run-loop-part-1/">runloop</a> in a few of the articles I have read but I had not had to deal with it directly until I started getting more and more unexpected behaviour in my tests.

This is best illustrated with some code.  Below is a simple Ember controller/object that creates a view in its constructor:
{%gist 2293396 %}
The above code creates an Ember model object on line 5 which is then bound to the view on line 7.  The view is then attached to the DOM on line 12. 

Below is my original failing test that I wrote:
{%gist 2293429 %}
I wrongly expected that the code on line 4 would create the controller and subsequently attach the view to the DOM immediately.  What was equally as perplexing was that the view appeared to be immediately appended onto the DOM when viewed in the browser.

I put this question out to the universe (twitter) and the universe (twitter) in all its majesty responded in kind.  I cannot actually remember who answered (apologies) but the response was along the lines of:
{%blockquote%}
Ember defers the DOM manipulation until <strong>later</strong> via the runloop.  You can make the run loop to execute immediately by calling <strong>Ember.run.end()</strong>
{%endblockquote%}
This advice was only half right, more on that later.

So what was this crazy man's talk of a runloop?  To understand what the **runloop** is, it is worth considering one of Ember's strengths or main selling points which is its **bindings** support.  A binding simply connects the properties of two objects in such a way that when one property changes, the value of the other one changes.  With Ember you would generally not change the DOM directly, you would instead make a change to the model and let the relevant bindings reflect this change.  This is MVC done properly with changes to view being reflected in the model via the observer pattern.  With backbone.js, you generally react to DOM events which is limited because it is easy for the model and view to become unsynchronised and tedious error prone code is needed to keep the two in sync. 

The runloop is the mechanism that ensures all bindings propagate data changes.  In the example I outlined above, I have the following binding that binds my model to the view:
{%codeblock%}
urlSearchBinding: 'controller.url_search'
{%endcodeblock%}
Then in my handlebars template, I am binding the value of an input field to the **search_url** property of the model:
{%codeblock%}
\{\{view Ember.TextField size="60" valueBinding="urlSearch.search_url"\}\}
{%endcodeblock%}
The upshot of this is that when a user enters text into the input filed, the **search_url** property of the model will change to that of the value attribute of the input field.  The reverse is also true, a change to the model will result in the input changing.  We could take this further and have another object whose property subsequently changes when the **search_url** property changes of our model and so on.  This is could turn into a performance overhead if the Ember runtime was constantly reacting to each binding as and when they change.  

##Behold the runloop##
The runloop is charged with batching all of these binding updates up in order to execute them all at once at the end of the runloop.  Something triggers the start of the runloop which is typically one of the many supported browser DOM events like click, mouseup, etc., etc., etc.  The runloop can also be triggered manually with code or from the expiration of a timer.  During the course of the runloop a number of bindings may have changed like in our example, we might change the value of a bound property:
{%codeblock%}
@url_search.set('search_url', 'http://thesoftwaresimpleton.com/')
{%endcodeblock%}
The change to the binding is not propagated immediately but is placed on a queue or deferred for later execution at the end of a running runloop.  When a runloop is triggered from a DOM event or manually in code, the bindings are not flushed until a certain point in the execution lifecycle of the runloop.  The runloop is also in charge of other things like executing expired timers.  

The important thing to remember here is that Ember waits until all the bindings have been propagated or flushed before updating any views which is why my code works fine when running in the browser but not in my test.  The runloop had not finished and the bindings had not been flushed so the view did not update.

It is possible to manually start a runloop at any time which is the answer to my initial confusion and will allow me to test with confidence from here on in.  A runloop can be manually started at any time by placing your code between an **Ember.run.begin()** and an **Ember.run.end()** statement.  Once **Ember.rub.end()** is called, you can be assured that all your binding changes will be propagated and any timers will be expired.  Only after **Ember.run.end()** has been called will any views be updated.  With this assurance, I can now test for elements in the DOM in my tests.  Below is how the now infamous failing test can be made to pass. 
{%gist 2303759 %}
On line 5, I am triggering the runloop via the convenience method **Ember.run** which accepts a function as an argument.  **Ember.run()** will take care of wrapping the passed in function between Ember.run.begin() and Ember.run.end().  I should not need to say that the test now passes.

The runloop is now incorporated into the majority of my tests where I ensure that the unit or critical part of the application that I want to test is running in the context of a runloop.  Below is an example of a test where I test my interaction with an Ember state manager:
{%gist 2304654 %}
I ensure that the send method of the state manager is wrapped in a run loop.
##Conclusion##
If I had not been writing tests, I am not sure if I would have discovered the runloop.  I think the runloop is an amazingly powerful concept and is used in many other frameworks and platforms.  I think one of the IPhone frameworks uses it in a similar fashion.  

Before I finish, I want to point out another trapdoor that I fell down before becoming better acquainted with the now infamous runloop.
My first attempt at dealing with the runloop is in the gist below where I just blindly added **Ember.run.end()** to my test on line 5 like so:
{%gist 2293513 %}
This is bad for a number of reasons.  Namely other tests will rely on Ember.run.end() being called from this test and as we all know, tests should be atomic and not rely on each other.  If I delete the test, the runloop no longer completes, also the runloop is being terminated unexpectedly for other tests.  This was another blind alley I gladly wandered down before deciding to do the research and write about it in this blog.

That is it for this post but if anybody with greater Ember knowledge than myself disagrees with any of the above then please, please correct me in the comments below.