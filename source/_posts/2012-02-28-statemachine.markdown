---
layout: post
title: "Model, View, StateMachine?"
date: 2012-02-28 13:11
comments: true
categories: JavaScript Ember
---
I have been using <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> in my day job recently for quite a few months for a thick client javascript application.  I have enjoyed the order and structure that the framework has brought to my javascript code.  I have also learned a lot of organisational and structural techniques by reading the book <a href="http://www.amazon.co.uk/JavaScript-Patterns-Stoyan-Stefanov/dp/0596806752/ref=sr_1_1?ie=UTF8&qid=1330435566&sr=8-1" href="_blank">JavaScript Patterns</a> recently.

Like most developers, I am working on a sure fire billion dollar side <a href="http://www.leadcapturer.com" target="_blank">project</a>.  In order to maintain the clear distinction between my day job and my side project, I have decided to use <a href="http://emberjs.com/">Ember.js</a> as my client side framework of choice.  I have been very impressed by what I have found so far.  I don't want to write too much about the differences between backbone and Ember or even too much about Ember as there is a wealth of information already out there but I will point out my preferences for using Ember.js over Backbone.js:

1. **Bindings -** Ember relies heavily on bindings as the core abstraction while backbone relies on raw DOM events.  A binding simply connects the properties of two objects so that whenever the value of one property changes, the value of the other one changes.  For example below is an Ember view I have defined:
{% gist 1932996 %}
On line 3 of the above gist, I have created a property called urlSearchBinding.  Ending a property with the postfix **Binding** tells Ember to create a binding that points to a property of the controller in which the view is defined called url_Search.  I can now reference this binding in my handlebars template like this:
{% gist 1933213 %}
On line 2 of the above gist, you can see that I am binding the value of the rendered input element to the *search_url* property of the *urlSearchBinding* that was defined on the view via the *valueBinding* property of the *Ember.TextField* control.  The upshot of this is that that whenever the value in the input changes the object that is bound on the view changes.  There is a lot more to bindings that you can read about in the <a href="http://ember-docs.herokuapp.com/#doc=Ember.Binding&src=false" target="_blank">Ember Docs</a>.  You can also set up <a href="http://ember-docs.herokuapp.com/#doc=Ember.Observable&src=false" target="_blank">observables</a> that help tie the Ember MVC pattern together.

2. **Out of the box Handlebars support - ** Ember takes a more opinionated view on which clientside templating library to use and comes already hooked up with <a href="http://www.handlebarsjs.com">Handlebars</a> support.  I use <a href="http://documentcloud.github.com/underscore/#template" target="_blank">underscore.js templating</a> with backbone and handlebars is by far the superior library.  I have also used JQuery templating which is my least favourite of the three I have tried.

##OK, but what about StateMachines?##
While looking into Ember I found an interesting Ember addon called the <a href="https://github.com/emberjs-addons/sproutcore-statechart" target="_blank">Ember Statechart</a>.  I have no idea why they called this a statechart and not a statemachine which is really what it is.  The <a href="http://en.wikipedia.org/wiki/Finite-state_machine" targe="_blank">state machine</a> pattern is a tried and trusted way of organising computer programs that has existed for a hell of a long time.  Simply put a state machine can be in only one of a finite number of states.  The machine is only in one state at a time;  the state at any current time is called the *current state*.  It can change from one state to another when initiated by a triggering event or condition, this is called a transition.  The classic example is a state machine that represents a lightbulb transitioning from the off state to the on state.  It is also possible to associate actions to a state:
 * **Entry Action - ** which is performed when entering a state.
 * **Exit Action -** which is performed when exiting a state.
In the ember statechart, these two actions or handlers manifest themselves as the *enterState* handler and the *exitState* handler.

##Show me some code##

##Routing##
<a href="https://github.com/DominikGuzei/ember-routing-statechart-example" target="_blank">routing example</a>