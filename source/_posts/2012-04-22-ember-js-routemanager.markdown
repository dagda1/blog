---
layout: post
title: "Ember.js, Routing with States"
date: 2012-04-22 20:43
comments: true
categories: JavaScript Ember
---
When I first started investigating <a target="_blank" href="http://emberjs.com/">Ember.js</a> I wrote a <a target="_blank" href="http://www.thesoftwaresimpleton.com/blog/2012/02/28/statemachine/">post</a> about an Ember addon called the <a href="http://docs.sproutcore.com/symbols/SC.State.html" target="_blank">SC.StateChart</a> that is more part of the old sproutcore framework than ember.  In that post, I wrote that I liked the *enterState* and *exitState* handlers and how they would be ideal for switching between views.  As of *Ember 0.9.6*, the **SC** namespace is no longer supported and I decided to refactor out anything that was Sproutcore related.  It was during this refactoring that I found a very similar object called the <a href="http://docs.emberjs.com/#doc=Ember.StateManager&src=false" target="_blank">Ember.StateMachine</a> that already exists in the Ember core code. It turns out that when an Ember StateMachine is composed of child **ViewState** state objects, the *StateManager* will take care of initialising and destroying currently displayed views as you transition between states.

The *Ember StateManager* is part of Ember's implementation of a <a href="http://en.wikipedia.org/wiki/Finite-state_machine" targe="_blank">finite state machine</a>.  Simply put a state machine can be in only one of a finite number of states.  The machine is only in one state at a time;  the state at any current time is called the *current state*.  It can change from one state to another when initiated by a triggering event or condition, this is called a transition.  The classic example is a state machine that represents a lightbulb transitioning from the **off** state to the **on** state.  

Let us flesh this out with some code.  Below is a StateManager that I am currently working on:
{% gist 2466587 %}
- On **line 1**, I am defining a definition from which a **StateManager** instance can be created from.  I am actually extending another definition called the **RouteManager** that extends **StateManager**.  The **RouteManager** is part of an excellent addon called the <a href="https://github.com/ghempton/ember-routemanager" target="_blank">ember-routemanager</a> that ties the StateMachine and a routing implementation together.  More on this later.
- On **line 3**, the **initialState** property is set to instruct the *StateManager* which state to transition to when an instance is created.  I am using the Ember path syntax to say that the initial state will be a *state* named **home** (line 11) that is a child of a parent state named **main** (line 6).
- On **line 6** is the definition of the previously mentioned parent **main** state. A StateManager is composed of child state objects.  In this example, the StateManager is composed of **ViewState** objects.  
- On **line 15**, is another state named *vault* that has a child state named *index*.

An **Ember.ViewState** object extends **Ember.State** and as the name suggests, this allows **Ember.View** objects to be associated with the parent state.  All child state objects in the above example are **ViewState** state objects and each **view** object has a *templateName* property that is a relative path to a <a href="http://www.handlebarsjs.com" target="_blank">handlebars</a> template.  

Anybody from a .NET background will wince at the name ViewState but I can assure you, there is no comparison whatsoever. 

When a StateManger is composed of instances of **ViewState** objects, the StateManager will interact with Ember's view system and manage which views are added and removed from the DOM based on the StateManager's current state.  One way of transitioning between states is to invoke the **goToState** method:
{% codeblock %}
@routeManager = WZ.ContentRouteManager.create()
@routeManager.start()

@routeManager.goToState 'vault.index'
{% endcodeblock %}
The above example will transition to the **index** state in the initial gist which is a child of the **vault** state.  Using the Ember path syntax allows us to transition to a child state in one statement, no matter how deep it is.  All views will be destroyed and attached to the DOM without you having to write any additional code.

The StateManager can also receive and route **action** messages to its states via the **send** message, you can checkout the docs to see this in action.

I find the *StateManager* very a very elegant solution that is bereft of tedious boilperplate code.  This is in stark contrast to Backbone.js where the user is left with both the architectural decision and the execution of how this should take place.  I mentioned one such solution in my <a href="http://www.thesoftwaresimpleton.com/blog/2011/11/13/backbone-js---lessons-learned/" target="_blank">Backbone.js - Lessons learned</a> post.

My first attempts at Backbone were littered with a lot of repetitive code like the following:
{% gist 2466780 %}
I never feel right about criticising oss code that someone has spent a lot of time developing but having worked with backbone for a number of months now, I do feel that there is too much boilerplate code required to glue everything together.  I have started using <a href="https://twitter.com/#!/derickbailey" target="_blank">Derick Bailey's</a> excellent <a href="https://github.com/derickbailey/backbone.marionette" target="_blank">backbone.marionette</a> recently with Backbone and you really need to use this if you are using Backbone regularly. 

##Nested Views##
When ViewState objects are nested like in the example below:
{% gist 2472193 %}
Each state will draw into the StateManager's **rootView** property (line 2) the rootElement property if you prefer that.  In the above example, the handlebars template in the **main** state's child view object will be compiled and attached to the DOM and the handlebars template defined in the **home** state's child view object will be compiled and attached to the DOM.

Below is how the **main** state's view and the **home** state's view look when they are rendered on the browser:
{%img /images/ember/mainview.png%}