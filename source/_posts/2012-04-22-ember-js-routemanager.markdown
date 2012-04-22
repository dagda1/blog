---
layout: post
title: "Ember.js, Routing with States"
date: 2012-04-22 20:43
comments: true
categories: JavaScript Ember
---
When I first started investigating <a target="_blank" href="http://emberjs.com/">Ember.js</a> I wrote a <a target="_blank" href="http://www.thesoftwaresimpleton.com/blog/2012/02/28/statemachine/">post</a> about an Ember addon called the SC.StateChart that is more part of the old sproutcore framework than ember.  In that post, I wrote that I liked the *enterState* and *exitState* handlers and how they would be ideal for switching between views.  What I did not realise was that Ember has a very similar object called the **Ember.StateMachine** which when composed of child **ViewState** objects will take care of initialising and destroying currently displayed views as you transition between states.  I will now flesh this out in the rest of this post.

The *Ember StateManager* is part of Ember's implementation of a <a href="http://en.wikipedia.org/wiki/Finite-state_machine" targe="_blank">finite state machine</a>.  Simply put a state machine can be in only one of a finite number of states.  The machine is only in one state at a time;  the state at any current time is called the *current state*.  It can change from one state to another when initiated by a triggering event or condition, this is called a transition.  The classic example is a state machine that represents a lightbulb transitioning from the **off** state to the **on** state.  

Let us flesh this out with some code.  Below is a StateManager that I am currently working on:
{% gist 2466587 %}
- On **line 1**, I am defining a definition from which a **StateManager** instance can be created from.  I am actually extending another definition called the **RouteManager** that extends **StateManager**.  The **RouteManager** is part of an excellent addon called the <a href="https://github.com/ghempton/ember-routemanager" target="_blank">ember-routemanager</a> that ties the StateMachine and a routing implementation together.  More on this later.
- On **line 3** I am stating which state the *StateManager* will initially transition to.  I am using the Ember path syntax to say that the initial state will be an object named **home** that is a child of an object named **main**.
- On **line 6** is the definition of the parent **main** state. A state manager is composed of child state objects.  In this example, the StateManager is composed of **ViewState** objects.  An **Ember.ViewState** extends **Ember.State** and as the name suggests allows Ember.View objects to be associated with the parent state.  

When a StateManger is composed of instances of ViewStates, the StateManager will interact with Ember's view system and manage which views are added and removed from the DOM based on the 