---
layout: post
title: "Ember.js - Model, View, StateMachine?"
date: 2012-02-28 13:11
comments: true
categories: JavaScript Ember
---
I have been using <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> in my day job recently for quite a few months for a thick client javascript application.  I have enjoyed the order and structure that the framework has brought to my javascript code.  I have also learned a lot of organisational and structural techniques by reading the book <a href="http://www.amazon.co.uk/JavaScript-Patterns-Stoyan-Stefanov/dp/0596806752/ref=sr_1_1?ie=UTF8&qid=1330435566&sr=8-1" href="_blank">JavaScript Patterns</a> recently.

Like most developers, I am working on a sure fire billion dollar side <a href="http://www.leadcapturer.com" target="_blank">project</a>.  In order to maintain the clear distinction between my day job and my side project, I have decided to use <a href="http://emberjs.com/">Ember.js</a> as my client side framework for this billion dollar baby.  I have been very impressed by what I have found so far.  I have also found an interesting new paradigm for clientside MV* that inspired me to write this post.  I don't want to say too much about the differences between backbone and Ember or even too much about Ember as there is a wealth of information already out there but I will point out my preferences for using Ember.js over Backbone.js:

- **Bindings -** Ember relies heavily on bindings as the core abstraction while backbone relies on raw DOM events.  A binding simply connects the properties of two objects so that whenever the value of one property changes, the value of the other one changes.  For example below is a simple Ember controller that contains a simple Ember view:
{% gist 1932996 %}
On line 7 is a property called *urlSearchBinding*. Ending a property with the postfix **Binding** tells Ember to create a binding that points to, in this context is a property of the controller called *url_Search* although it could be a property of any available object.  I can now reference this binding in my handlebars template like this:
{% gist 1933213 %}
On line 2 of the above gist, you can see that I am binding the value of the rendered input element to the *search_url* property of the *urlSearchBinding* that was defined on the view via the *valueBinding* property of the *Ember.TextField* control.  The upshot of this is that that whenever the value attribute of the input changes, the urlSearch's search_url property  changes.  I must prefer this approach than having to pull values out of the DOM.  There is a lot more to bindings that you can read about in the <a href="http://ember-docs.herokuapp.com/#doc=Ember.Binding&src=false" target="_blank">Ember Docs</a>.  You can also set up <a href="http://ember-docs.herokuapp.com/#doc=Ember.Observable&src=false" target="_blank">observables</a> that help tie the whole Ember MVC pattern together.

- **Out of the box Handlebars support - ** Ember takes a more opinionated view on which clientside templating library to use and comes already hooked up with <a href="http://www.handlebarsjs.com">Handlebars</a> support.  I use <a href="http://documentcloud.github.com/underscore/#template" target="_blank">underscore.js templating</a> with backbone and handlebars is by far the superior library.  I have also used JQuery templating which is my least favourite of the three I have tried.

##OK, but what about StateMachines?##
While looking into Ember I found an interesting Ember addon called the <a href="https://github.com/emberjs-addons/sproutcore-statechart" target="_blank">Ember Statechart</a>.  I have no idea why they called this a statechart and not a statemachine which is really what it is.  The <a href="http://en.wikipedia.org/wiki/Finite-state_machine" targe="_blank">state machine</a> pattern is a tried and trusted way of organising computer programs that has existed for a hell of a long time.  Simply put a state machine can be in only one of a finite number of states.  The machine is only in one state at a time;  the state at any current time is called the *current state*.  It can change from one state to another when initiated by a triggering event or condition, this is called a transition.  The classic example is a state machine that represents a lightbulb transitioning from the off state to the on state.  It is also possible to associate actions to a state:

- **Entry Action - ** which is performed when entering a state.
- **Exit Action -** which is performed when exiting a state.

In the ember statechart, these two actions or handlers manifest themselves as the *enterState* handler and the *exitState* handler.  These handlers are ideal for a complex UI interactions.  At a very basic level, you could use this paradigm to great affect to switch between tabs in a tab control for example.  The entry and exit state handlers provide a perfect place to initialise the views, persist state, hide and show elements or dispose of views and resources.  This seems a perfect match for a rich client javascript application.

##Show me some code##
Below is a very simple example of a statechart:
{% gist 1934749 %}
Now let me breakdown the code

- On **line 1** the statechart class (for want of a better word) is defined by extending *SC.Statechart*.  It is worth noting that extend does not create the instance but only defines the class.
- On **line 2** the *monitorIsActive* property provides you with debug info.  I have yet to use this.
- On **line 4** the *substatesAreConcurrent* property indicates if the state's immediate substates are to be concurrent (orthogonal) to each other. 

##Routing##
<a href="https://github.com/DominikGuzei/ember-routing-statechart-example" target="_blank">routing example</a>