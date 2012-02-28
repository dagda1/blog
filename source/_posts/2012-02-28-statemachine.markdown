---
layout: post
title: "Model, View, StateMachine?"
date: 2012-02-28 13:11
comments: true
categories: JavaScript Ember
---
I have been using <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> in my day job recently for quite a few months in a rich client javascript application.  I have enjoyed the order and structure that the framework has brought to my javascript code.  I have also learned a lot of organisational and structural techniques by reading the book <a href="http://www.amazon.co.uk/JavaScript-Patterns-Stoyan-Stefanov/dp/0596806752/ref=sr_1_1?ie=UTF8&qid=1330435566&sr=8-1" href="_blank">JavaScript Patterns</a> recently.

Like most developers, I am working on a sure fire billion dollar side <a href="http://www.leadcapturer.com" target="_blank">project</a>.  In order to maintain the clear distinction between my day job and my side project, I have decided to use <a href="http://emberjs.com/">Ember.js</a> as my client side framework of choice.  I have been very impressed by what I have found so far.  I don't want to write too much about the differences between backbone and Ember or even too much about Ember as there is a wealth of information already out there but I will point out my preferences for using Ember.js over Backbone.js:

1. **Bindings -** Ember relies heavily on bindings as the core abstraction while backbone relies on raw DOM events.  A binding simply connects the properties of two objects so that whenever the value of one property changes, the value of the other one changes.  For example below is an Ember view I have defined:
{% gist 1932996 %}
On line 3 of the above gist, I have created a property called urlSearchBinding.  Ending a property with binding tells Ember to create a binding to a property of the controller in which the view is defined called urlSearch.  I can now reference this binding in my handlebars template like this:
{% gist 1933213 %}