---
layout: post
title: "Backbone.js - Lessons Learned"
date: 2011-11-13 13:14
comments: true
categories: JavaScript Backbone.js
---
I have been using <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> for a few months now and I have to say that I have found the learning curve to be a steep one.  I am not sure if this is an inherent framework smell or that it just takes time to learn anything new with complexity.  There are a number of ways of doing anything in Backbone.js and I think the framework could do with some tightening up to stop users having to roll their own custom implementations for common scenarios such as disposing of views.  

For those that have been sleeping under a rock recently, <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> is a client side MVC framework that should not really require any introduction.  The client side MVC framework seems to be a bit of a du jour thing at the moment but I think it is important to have some sort of organisational structure around JavaScript or the looseness of the language can quickly turn any javascript code into the now infamous ball of mud.  Backbone.js is one way of achieving such a conformity.

The point of this post is to solidify where I currently at with Backbone.js and to welcome any comments from my readership of 10 or so readers as to a better and brighter land.  I must also warn you that I use coffeescript instead of pure javascript and if this offends then please read no further.

## Scenario

The scenario I want to illustrate is a simple one, I want to show how I transition from the state outlined below were a user selecmi.jpgts from a list of what are called in the application context, business units:
{% img /images/backbone/bu.png %}
To a more detailed report illustrated in the image below:
{% img /images/backbone/mi.png %}
The user can go back and forward between these views and drill down into further views of the information but I want to concentrate on this simple scenario.  It should be noted that this is simply readonly and I am not going to delve into modifying models and collections in this post.

###  Gotcha 1 - Avoid holding a reference of one view in another
In the scenario I have listed, I need some way to gather the user's checkbox selection and pass these values to the view that renders the table and chart of results.  My attempt at this in other applications was to let one of the views hold a reference to another and pass the information that way.   This type of naive implementation might look like this:
{% gist 1362192 %}
This felt very wrong as there is now a coupling between the views. Thankfully somebody on twitter kindly pointed me in the direction of this <a href="http://lostechies.com/derickbailey/2011/07/19/references-routing-and-the-event-aggregator-coordinating-views-in-backbone-js/" target="_blank">post</a> from <a href="https://twitter.com/#!/derickbailey" target="_blank">Derek Bailey</a> who is a wealth of information on Backbone.js. 

To summarise the above post, instead of holding a reference to the view we want to render the collection from, we instead create a central object that manages the raising of events and the subscribers for those events.  Martin Fowler calls this the <a href="http://martinfowler.com/eaaDev/EventAggregator.html">Event Aggregator</a> pattern.  This serves the purpose of decoupling the views from each other.  The entire listing of the event aggregator is on line 3 of the following gist:
{% gist 1362205 %}
We are taking advantage of the way Backbone already handles events.  The Backbone events object is a module that can be mixed in with any object, giving the object the ability to bind and trigger custom named events.  On lines 4 and 5, we are making sure that any of the views that want to publish or subscribe to events have a reference to the **vent** object.  With this in place, I can simply raise an event from the BusinessUnitView and pass the newly created collection as an event argument.  Below is the BusinessUnitView refactored to take advantage of the event aggregator:
{% gist 1362495 %}
The main difference is on line 2 where we are holding a reference to our extended Backbone.js **Events** object and on line 3 where the **bind** method of the Events object is used to specify a callback that will be invoked whenever the event (specified by the string identifier) is fired.  In this case we want to want this code to be ran whenever the back button of the second view is clicked.  Lines 20 to 23 show the 