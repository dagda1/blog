---
layout: post
title: "Testing Ember, A.K.A. How I Learned to Love the Runloop and Stop Worrying"
date: 2012-04-03 07:55
comments: true
categories: JavaScript Ember
---
I am currently using <a href="http://emberjs.com/">Ember.js</a> for my soon to be billion dollar side project and I have ran into some interesting behaviour while driving the development of the front end code test first.  I am using the excellent javascript bdd library <a href="http://pivotal.github.com/jasmine/" target="_blank">Jasmine</a> as my test runner.  I have also been using the excellent <a href="https://github.com/netzpirat/guard-jasmine">guard-jasmine</a> gem to automatically run my tests when any files are modified which really is a great experience.  

I had vaguely heard of the Ember <a href="http://blog.sproutcore.com/the-run-loop-part-1/">runloop</a> in a few articles I had read but I did not have to deal with it directly until I started getting more and more unexpected behaviour in my tests.

This is best illustrated with some code.  Below is a simple Ember controller/object that creates a view in its constructor:
{%gist 2293396 %}
The above code creates an Ember model object on line 5 which is then bound to the view on line 7.  The view is then seemingly attached to the DOM on line 12.  This should all happen when I **create** an instance of the UrlSearch class.

Below is my original failing test that I wrote:
{%gist 2293429 %}
I wrongly expected that the code on line 4 would create the controller and subsequently attach the view to the DOM immediately.  What was equally as perplexing was that the view was appended onto the DOM when viewed in the browser.

I put this question out to the universe (twitter) and the universe (twitter) in all its majesty responded in kind.  I cannot actually remember who answered (apologies) but the response was along the lines of:
{%blockquote%}
Ember defers the DOM manipulation until <strong>later</strong> via the runloop.  You can make the run loop to execute immediately by calling <strong>Ember.run.end()</strong>
{%endblockquote%}
This advice was only half right, more on that later.

So what was this crazy man's talk of a runloop?  To understand what the **runloop** is, it is worth considering one of Ember's strengths or main selling points for me which are **bindings**.  A binding simply connects the properties of two objects in such a way that when one property changes, the value of the other one changes.  With Ember you would not generally change the DOM directly, you would instead make a change to the model and let the relevant bindings reflect this change.  With backbone.js, you generally react to DOM events which is limited because it is easy for the model and view to become unsynchronised and tedious error prone code is needed to keep the two in sync. 

The runloop is mechanism that ensures all bindings propagate data changes.  In the example I outline above, I have the following binding that binds my model to the view:
{%codeblock%}
urlSearchBinding: 'controller.url_search'
{%endcodeblock%}
Then in my handlebars template, I am binding the value of an in put field to the **search_url** property of the model:
{%codeblock%}
urlSearchBinding: 'controller.url_search'
{%endcodeblock%}

Ember.run.begin(), Ember.run.end()

So of course I blindly added **Ember.run.end()** to my test on line 5 like so:
{%gist 2293513 %}
