---
layout: post
title: "Backbone.js - Lessons Learned"
date: 2011-11-13 13:14
comments: true
categories: JavaScript Backbone.js
---
I have been using <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> for a few months now and I have found the learning curve to be a steep one.  As is the case with any flexible framework, there are a number of ways of doing anything in Backbone.js and I think the framework could do with some tightening up to stop users having to roll their own custom implementations for common scenarios such as disposing of views.

For those that have been sleeping under a rock recently, <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> is a client side MVC framework that should not really require any introduction.  The client side MVC framework seems to be a bit of a du jour thing at the moment but I think it is important to have some sort of organisational structure around JavaScript or the looseness of the language can quickly turn any javascript code into the now infamous ball of mud.  Backbone.js is one way of achieving such a conformity.

The point of this post is to solidify where I my current thinking is with Backbone.js and to welcome any comments from my readership of 10 or so readers that might point me to a better and brighter land.  I must also warn you that I will be using coffeescript for the code examples instead of pure javascript and if this offends, please read no further.

## Scenario

The scenario I want to illustrate is a simple one, I want to show how I transition from the state outlined below were a user selects from a list of what are called in the application context, business units:
{% img /images/backbone/bu.png %}
To a more detailed report illustrated in the image below:
{% img /images/backbone/mi.png %}
The user can go back and forward between these views and drill down into further views of the information but I want to concentrate on this simple scenario.

###  Gotcha 1 - Don't Have Views That Reference Each Other a.k.a. Coupling
In the scenario I have listed, I need some way to gather the user's checkbox selection and pass these values to the view that renders the table and chart of results.  My attempt at this in other applications was to let one of the views hold a reference to another and pass the information that way.   This type of naive implementation might look like this:
{% gist 1362192 %}
This felt very wrong as there is now a coupling between the views. Thankfully somebody on twitter kindly pointed me in the direction of this <a href="http://lostechies.com/derickbailey/2011/07/19/references-routing-and-the-event-aggregator-coordinating-views-in-backbone-js/" target="_blank">post</a> from <a href="https://twitter.com/#!/derickbailey" target="_blank">Derek Bailey</a> who is a wealth of information on Backbone.js. 

To summarise the above post, instead of holding a reference to the view we want to render the collection from, we instead create a central object that manages the raising of events and the subscribers for those events.  Martin Fowler calls this the <a href="http://martinfowler.com/eaaDev/EventAggregator.html">Event Aggregator</a> pattern.  This serves the purpose of decoupling the views from each other.  The entire listing of the event aggregator is on line 3 of the following gist:
{% gist 1362205 %}
We are taking advantage of the way Backbone already handles events.  The Backbone **Events** object is a module that can be mixed in with any object, giving the object the ability to bind and trigger custom named events.  On lines 4 and 5, we are making sure that any of the views that want to publish or subscribe to events have a reference to the **vent** object.  With this in place, I can simply raise an event from the BusinessUnitView and pass the newly created collection as an event argument.  Below is the BusinessUnitView refactored to take advantage of the event aggregator:
{% gist 1362495 %}
The main difference is on line 3 where the view obtains a reference to our extended Backbone.js **Events** object and on line 20 where our custom **Events** object is used to **trigger** a custom event that is uniquely identified by a string.  We are using the convention of a colon separated string to namespace the events and avoid collisions.  Of course we are making the presumption that there is a similarly named event somewhere in our code and of course there is.  Below is the code listing for the ManagementReportView that subscribes to the event listed above:
{%gist 1365243 %}
On line 3 of the above gist, we are using the **bind** method of the Events object to specify a callback that will be triggered when the callback is invoked and any optional arguments can be passed to the callback which in this instance is the newly created businessunits backbone collection object. On line 7 we define the callback.  The **load** callback on line 7 is declared using the fat arrow **=>** syntax.  Coffeescript will take care of binding methods to *this* if you declare a function using the fat arrow **=>** instead of the skinny one **->**.  This means you don't need to use **_.bindAll** which can be quite tedious and error prone.  On line 8 of the load callback we are setting the internal collections reference to the argument raised from the triggered event and line 9 brings us to our next gotcha:

###  Gotcha 2 - Handle Backbone Model Events and not DOM Events Wherever Possible
Part of my reason for warming to backbone is that it brings conformity and form to my JavaScript/Coffeescript.  It is all to easy for traditional browser javascript to descend into a sea of DOM event handlers.  Wherever possible, I have my views respond to model or collection events.  On line 9 of the above gist, I am adding an event handler to the **reset** event which is fired whenever a backbone collection is replaced with a bulk insert.  On line 10 we are calling the built in **fetch** method of the backbone collection to retrieve the data from the remote server.  When the data returns successfully from the remote server, the *reset* event is fired and we have bound the render event as a callback to this event on line 16.  You can subscribe to events that are fired whenever elements are added or removed from a collection or when an individual model changes and you should get accustomed to each of these behaviours.  Retrieving the data from the remote server brings me to gotcha 3 and in effect the render method:

### Gotcha 3 - SRP - Keep Your Views Skinny
It is all to easy to end up with bloated views that do way to much and my first few attempts at backbone suffered from this.  I've seen code that make calls to the remote server in the view and this is wrong, let the collection/model do the remote call.  Below is the listing of the BusinessUnits object:
{%gist 1365331 %}
As this is javascript (coffeescript really) we can override any of the built in backbone collection methods like I am doing with the url method on line 6 but you can if required override **fetch** method for example for tighter control.  

The inner workings of the render method brings me to my last and most troublesome gotcha.

###  Gotcha 4 - Drive A Stake Through Your Zombies
I cannot take the credit for the excellent zombie analogy, this again goes to Derek Bailey and a combination of this <a href="http://lostechies.com/derickbailey/2011/09/15/zombies-run-managing-page-transitions-in-backbone-apps/" target="_blank">Post</a> and this <a href="http://stackoverflow.com/questions/7567404/backbone-js-repopulate-or-recreate-the-view/7607853#7607853">Stackoverflow</a> question and answer session.  I have combined these posts into how I now handle this common scenario.

If we refer back to the screenshot I showed at the start:
{% img /images/backbone/mi.png %}
I want to render a subview for every row in the above screenshot that has a cross or a tick image.  The gotcha outlined here is that I need to dispose of these views whenever the **Back** button on the top right is clicked.  If I don't dispose of the subviews, new subviews will be added every time the user transitions to this view with a new collection selection.  Multiple event handlers and all round chaos will occur and I have been there.

Let us take a look again at the render method:
{% codeblock%}
render: =>
  self = @

  @collection.each (item, counter) ->
    self.renderView(self.el, 'append', new ReportBusinessUnitView(model: item, vent: self.vent))
{%endcodeblock%}
Here we are simply iterating over our collection and appending onto this view's element a new subview for each element of the collection.  We are actually calling a method named **renderView** of our own **BaseView** class that extends the Backbone.View class.  More on that later but for completeness, below is the ReportBusinessUnitView class:
{%gist 1365460 %}
I've also found underscore's templating to be much faster than JQuery's which I did start out using, below is the underscore template:
{% gist 1365477 %}
So that code will take care of iterating over our collection and attaching the subviews to our parent view.  But if the user pressed back and then chose another selection and transitioned to this view again, we would add more subviews to this parent view.  We need someway of keeping track of and disposing of our subviews.