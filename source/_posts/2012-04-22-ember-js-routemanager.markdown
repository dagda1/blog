---
layout: post
title: "Ember.js - Routing with States"
date: 2012-04-22 20:43
comments: true
categories: JavaScript Ember
---
When I first started investigating <a target="_blank" href="http://emberjs.com/">Ember.js</a> I wrote a <a target="_blank" href="http://www.thesoftwaresimpleton.com/blog/2012/02/28/statemachine/">post</a> about an Ember addon called the <a href="http://docs.sproutcore.com/symbols/SC.State.html" target="_blank">SC.StateChart</a> that is really part of the old Sproutcore framework.  In that post, I wrote that I liked the *enterState* and *exitState* handlers of the *StateChart* and how they would be ideal for switching between views.  As of *Ember 0.9.6*, the **SC** namespace is no longer supported and I decided to refactor/remove anything that was Sproutcore related.  It was during this refactoring that I found a very similar object called the <a href="http://docs.emberjs.com/#doc=Ember.StateManager&src=false" target="_blank">Ember.StateManager</a> that already exists in the Ember core code. It turns out that the Ember StateManager is a very elegant solution to transitioning between views and attaching and removing views from the DOM.

The *Ember StateManager* is part of Ember's implementation of a <a href="http://en.wikipedia.org/wiki/Finite-state_machine" targe="_blank">finite state machine</a>.  Simply put a state machine can be in only one of a finite number of states.  The machine is only in one state at a time;  the state at any current time is called the *current state*.  It can change from one state to another when initiated by a triggering event or condition, this is called a transition.  The classic example is a state machine that represents a lightbulb transitioning from the **off** state to the **on** state.  

Let us flesh this out with some code.  Below is a StateManager that I am currently working on:
{% gist 2466587 %}
- On **line 1**, I am defining a definition from which a **StateManager** instance can be created from.  I am actually extending another definition called the **RouteManager** that extends **StateManager**.  The **RouteManager** is part of an excellent addon called the <a href="https://github.com/ghempton/ember-routemanager" target="_blank">ember-routemanager</a> that ties the *StateManager* and a routing implementation together.  More on this later.
- On **line 3**, the **initialState** property is set to instruct the *StateManager* which state to transition to when an instance of the *StateManager* is created.  I am using the Ember path dot syntax to say that the initial state will be a *state* named **home** (line 11) that is a child of a parent state named **main** (line 6).
- On **line 6** is the definition of the previously mentioned parent **main** state. A StateManager is composed of child state objects.  In this example, the StateManager is composed of **Ember.ViewState** objects. **Ember.ViewState** objects can contain **Ember.View** objects (as you would expect). 
- On **line 15**, is another state named **vault** that has a child state named **index** (line 20).

An **Ember.ViewState** object extends **Ember.State** and as the name suggests, **Ember.View** objects are associated with the **Ember.ViewState** .  All child state objects in the above example are **ViewState** state objects and each **view** has a **templateName** property that is a relative path to a <a href="http://www.handlebarsjs.com" target="_blank">handlebars</a> template.  

Anybody from a .NET background will wince at the name ViewState but I can assure you, there is no connnection whatsoever. 

When a StateManger is composed of instances of **ViewState** objects, the StateManager will interact with Ember's view system and manage which views are added and removed from the DOM based on the StateManager's current state.  One way of transitioning between states is to invoke the **goToState** method:
{% codeblock %}
@routeManager = WZ.ContentRouteManager.create()
@routeManager.start()

@routeManager.goToState 'vault.index'
{% endcodeblock %}
The above example will transition to the **index** state which is a child of the **vault** state.  Using the Ember path syntax allows you to transition to a child state in one statement, no matter how deep it is.  When you transition from one ViewState to another, the view of the **currentState** is destroyed and removed from the DOM as you **exit** the state and the view of the ViewState you are transitioning to is created and attached to the DOM as you **enter** the new state.

The StateManager can also receive and route **action** messages to its states via the **send** message, which you can read about in the <a href="http://docs.emberjs.com/#doc=Ember.StateManager&src=false" target="_blank">Ember docs</a>.

I find the StateManager a very elegant solution that is bereft of tedious boilerplate code.  This is in stark contrast to Backbone.js where the user is left with both the architectural decision and the execution of how this should take place.  I mentioned one such solution in my <a href="http://www.thesoftwaresimpleton.com/blog/2011/11/13/backbone-js---lessons-learned/" target="_blank">Backbone.js - Lessons learned</a> post.

My first attempts at Backbone were littered with a lot of repetitive code like the following:
{% gist 2466780 %}
In Backbone, I think there is too much boilerplate code required to glue everything together.  I have started using <a href="https://twitter.com/#!/derickbailey" target="_blank">Derick Bailey's</a> excellent <a href="https://github.com/derickbailey/backbone.marionette" target="_blank">backbone.marionette</a> recently with Backbone and you really need to use this if you are using Backbone regularly. 

##Nested ViewStates##
When ViewState objects are nested like in the example below:
{% gist 2472193 %}
Both the StateManager's **currentState** and any child states of the **currentState** will draw into the StateManager's **rootView** property (line 2).  In the above example, the RouteMangaer will initially transition to the path defined in the **initialState** property which in this case is the **main.home** path which points to the **home** state on line 11.  This means that the handlebars template declared with the **templateName** property in both the **main** state's child view object will be compiled and attached to the DOM **and** the handlebars template defined in the **home** state's child view object will be compiled and attached to the DOM.

Below is how the **main** state's view and the **home** state's view look when they are rendered onto the browser:
{%img /images/ember/mainview.png%}
You can get quite inventive and compose reusable partial views as and when you need them. I am setting the **rootView** property which means I can just layer views onto the body element of the html document but you can also specify a **rootElement** property:
{%codeblock%}
rootElement: '#some-other-element'
{%endcodeblock%}
##Routing##
This brings us nicely on to routing, as I mentioned earlier, I am using the excellent <a href="https://github.com/ghempton/ember-routemanager" target="_blank">ember-routemanager</a> addon that ties the StateManager to a routing implementation.  This addon allows you to create **RouteManager** instances that derive from the now infamous Ember StateManager.  Below is a reminder of how I am extending the RouteManager:

{%gist 2466587 %}
It is worth noting that I am not creating an instance of the RouteManager but merely extending it which will allow me to create an instance on application start up or create instances in my jasmine specs.

What you should note from the above gist is that I am defining a **route** property on each child state object (lines 7, 12, 16 and 21).  This allows me to use a combination of the route property and the nesting of the child states to define client routes that will transition to the required states and attach and remove the views from the DOM.

Below is how I create an instance of my derived **RouteManager**:
{% codeblock %}
@routeManager = WZ.ContentRouteManager.create()
@routeManager.start()
{% endcodeblock %}
The **start** method will start the RouteManger listening for location changes.  This allows me to define routes either as the now familiar hash links:
{% codeblock %}
<!--Transition to the vault.new state -->
<li><a href="#vault/new">New</a></li>
{% endcodeblock %}
Or I can programatically set the **location** property which is useful for testing:
{% codeblock %}
@routeManager.set 'location', 'vault/index'
{% endcodeblock %}
All that you would expect from a routing frameworks seems covered, below is an example of a route with a dynamic parameter:
{% codeblock %}
route: 'vault/:exerciseid'
{% endcodeblock %}
Setting the browser location to **#vault/5** would map to the vault state with an exerciseid of 5.  Wildcards and regular expressions are also covered:
{% codeblock %}
route: /(\d{4})-(\d{2})-(\d{2})/
{% endcodeblock %}

##Conclusion##
I got quite excited as I explored this approach (I should get out more).  This is a real productivity boom that could be further enhanced by having the **route** inferred using convention over configuration by dynamically creating the route with respect to the state's nesting.  The RouteManager/StateManager/StateMachine is a very elegant solution for transitioning between views that exists in many other smart client UI frameworks.  Being a web developer, I just have not come across it before.

##Testing the RouteManager##
I ran into a few problems when testing the RouteManager.  Below are some initial specs that now all pass:
{% gist 2473697 %}
One of the problems I had was clearing the **rootView** property and disposing of the **RouteManager**  after each spec.  This is  solved in the **afterEach** method on lines 10 to 16 of the above gist where I am removing each view from the DOM and destroying the StateManger.

Please feel free to add any comments below.