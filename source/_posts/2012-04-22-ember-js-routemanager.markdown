---
layout: post
title: "Ember.js, Routing with States"
date: 2012-04-22 20:43
comments: true
categories: JavaScript Ember
---
When I first started investigating <a href="http://emberjs.com/">Ember.js</a> I wrote a <a href="http://www.thesoftwaresimpleton.com/blog/2012/02/28/statemachine/">post</a> about an Ember addon called the SC.StateChart that is more part of the old sproutcore framework than ember.  In that post, I wrote that I liked the *enterState* and *exitState* handlers and how they would be ideal for switching between views.  What I did not realise was that Ember has a very similar object called the **Ember.StateMachine** which when composed of child **ViewState** objects will take care of initialising and destroying currently displayed views as you transition between states.  I will now flesh this out in the rest of this post.

The *Ember StateManager* is part of Ember's implementation of a <a href="http://en.wikipedia.org/wiki/Finite-state_machine" targe="_blank">finite state machine</a>.  Simply put a state machine can be in only one of a finite number of states.  The machine is only in one state at a time;  the state at any current time is called the *current state*.  It can change from one state to another when initiated by a triggering event or condition, this is called a transition.  The classic example is a state machine that represents a lightbulb transitioning from the off state to the on state.  It is also possible to associate actions to a state, for example:

- **Entry Action - ** which is performed when entering a state.
- **Exit Action -** which is performed when exiting a state.
Let us flesh this out with some code

Below is a StateManager that I am currently working on:
{%gist 2466587%}
