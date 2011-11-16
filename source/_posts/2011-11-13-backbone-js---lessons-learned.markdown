---
layout: post
title: "Backbone.js - Lessons Learned"
date: 2011-11-13 13:14
comments: true
categories: JavaScript Backbone.js
---
I have been using <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> for a few months now and I have found the learning curve to be a steep one.  As is the case with any flexible framework, there are a number of ways of doing anything in Backbone.js and I think the framework could do with some tightening up to stop users having to roll their own custom implementations for common scenarios such as disposing of views.

For those that have been sleeping under a rock recently, <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> is a client side MVC framework that should not require any introduction.  The client side MVC framework seems to be a bit of a du jour thing at the moment but I think it is important to have some sort of organisational structure around JavaScript or the looseness of the language can quickly turn any javascript code into the now infamous ball of mud.  Backbone.js is one way of achieving such a conformity.

The point of this post is to solidify where my current thinking is with Backbone.js and to welcome any comments from my readership of 10 or so readers that might point me to a better and brighter land.  I must also warn you that I will be using coffeescript for the code examples instead of pure javascript and if this offends, please read no further.

## Scenario

The scenario I want to illustrate is a simple one, I want to show how I transition from the state outlined below were a user selects from a list of what are called in the application context, business units:
{% img /images/backbone/bu.png %}
To a more detailed report illustrated in the image below:
{% img /images/backbone/mi.png %}
The user can go back and forward between these views and drill down into further views of the information but I want to concentrate on this simple scenario outlined above in the two screenshots.

###  Gotcha 1 - Don't Have Views That Reference Each Other a.k.a. Coupling
In the scenario I have listed, I need some way to gather the user's checkbox selection and pass these values to another view that renders the table and chart of results.  My first few attempts at this in other applications was to let one of the views hold a reference to another and pass the information that way.   An example of this type of naive implementation might look something like this:
{% gist 1362192 %}
This felt very wrong as there is now a coupling between the views. Thankfully somebody on twitter kindly pointed me in the direction of this <a href="http://lostechies.com/derickbailey/2011/07/19/references-routing-and-the-event-aggregator-coordinating-views-in-backbone-js/" target="_blank">post</a> from <a href="https://twitter.com/#!/derickbailey" target="_blank">Derek Bailey</a> who is a wealth of information on Backbone.js. 

The post outlines an alternative to creating and coupling one view in another like in the example above with the ManagementReportView.  We instead create a central object that manages the raising of events and the subscribers for those events.  Martin Fowler calls this the <a href="http://martinfowler.com/eaaDev/EventAggregator.html">Event Aggregator</a> pattern.  This serves the purpose of decoupling the views from each other.  The entire listing of the event aggregator is on line 3 of the following gist:
{% gist 1362205 %}
We are taking advantage of the way Backbone already handles events.  The Backbone **Events** object is a module that can be mixed in with any object, giving the object the ability to bind and trigger custom named events.  On lines 4 and 5, we are making sure that any of the views that want to publish or subscribe to events have a reference to the **vent** object.  With this in place, I can simply raise an event from the BusinessUnitView and pass the newly created collection as an event argument.  Below is the BusinessUnitView refactored to take advantage of the event aggregator:
{% gist 1362495 %}
The first addition is on line 3 where the view obtains a reference to our extended Backbone.js **Events** object and on line 20 where the custom **Events** object is used to **trigger** a custom event that is uniquely identified by a string.  We are using the convention of a colon separated string (**reportingbusinessunitview:load**) to namespace the events and avoid collisions.  Below is the code listing for the ManagementReportView that subscribes to the event listed above:
{%gist 1365243 %}
On line 3 of the above gist, we are using the **bind** method of the Events object to specify a callback that will be triggered when the event is raised and any optional arguments can be passed to the callback which in this instance is the newly created businessunits backbone collection that we created from the user input in the previous view. On line 7 we define the callback event handler.  The **load** callback on line 7 is declared using the fat arrow **=>** syntax.  Coffeescript will take care of binding methods to *this* if you declare a function using the fat arrow **=>** instead of the skinny one **->**.  This means you don't need to use the underscore **_.bindAll** method as I would in javascript which can be quite tedious and relies on your diligence.  One of the many reasons why you should be using coffeescript.  On line 8 of the load callback we are setting the view's collections reference to the argument raised from the triggered event and line 9 brings us to our next gotcha:

###  Gotcha 2 - Handle Backbone Model Events and not DOM Events Wherever Possible
Part of my reason for warming to backbone is that it brings conformity and organisation to my JavaScript/Coffeescript.  It is all too easy for traditional browser based javascript to descend into a sea of DOM event handlers.  Wherever possible, I have my views respond to backbone model or collection events.  On line 9 of the above gist, I am adding an event handler to the **reset** event which is fired whenever a backbone collection is replaced with a bulk insert.  On line 10 we are calling the built in **fetch** method of the backbone collection to retrieve the data from the remote server.  When the data returns successfully from the remote server, the *reset* event is fired and we have bound the render event as a callback to this event on line 16.  You can subscribe to events that are fired whenever elements are added or removed from a collection or when an individual model's attributes change and you should get accustomed to all of these behaviours.  Retrieving the data from the remote server brings me to gotcha 3 and in effect the render method:

### Gotcha 3 - SRP - Keep Your Views Skinny
It is all to easy to end up with bloated views that do way to much and my first few attempts at backbone suffered from this.  I've seen code that make calls to the remote server in the view and this is wrong. Let the collection/model do the remote call and subscribe to an appropriate backbone collection/model event.  Below is the listing of the BusinessUnits object:
{%gist 1365331 %}
As this is javascript (coffeescript really) we can override any of the built in backbone collection methods like I am doing with the url method on line 6 but you can if required override **fetch** or **parse** methods for tighter control.  

It is worth noting that the render method creates further subviews to attach to the parent view's element. When refactoring backbone code, you tend to split your views up into finer detail or subviews to further inforce SRP.

The inner workings of the render method brings me to my last and most troublesome gotcha.

###  Gotcha 4 - Kill All Your Zombies.....Dead
I cannot take the credit for the excellent zombie analogy or in fact any of the revelations in this post, this again goes to Derek Bailey and a combination of this <a href="http://lostechies.com/derickbailey/2011/09/15/zombies-run-managing-page-transitions-in-backbone-apps/" target="_blank">post</a> and this <a href="http://stackoverflow.com/questions/7567404/backbone-js-repopulate-or-recreate-the-view/7607853#7607853">Stackoverflow</a> question and answer session.  I have combined the contents of the above 2 links into how I now handle this common scenario.

If we refer back to the screenshot I showed at the start:
{% img /images/backbone/mi.png %}
I want to render a subview for every row in the above screenshot that has a cross or a tick image in the table columns.  The gotcha outlined here is that I need to dispose of these views whenever the **Back** button on the top right is clicked.  If I don't dispose of the subviews, new and additional subviews will be added every time the user transitions to this view with a new collection selection.  Multiple event handlers and general round chaos will occur and I have been there and it can be distressing to the uninitiated.  There will be a number of views that are not attached to any element floating about in memory which gives them their zombie status.

Let us take a look again at the render method:
{% codeblock%}
render: =>
  self = @

  @collection.each (item, counter) ->
    self.renderView(self.el, 'append', new ReportBusinessUnitView(model: item, vent: self.vent))
{%endcodeblock%}
Here we are simply iterating over our collection and appending onto this view's element a new subview for each element of the collection.  We are actually calling a method named **renderView** of our own **BaseView** class that extends the Backbone.View class.  More on that later but for completeness, below is the ReportBusinessUnitView class:
{%gist 1365460 %}
I've also found underscore's templating to be much faster than JQuery's templating engine which I initially started out using. Below is the underscore template:
{% gist 1365477 %}
The code will take care of iterating over our collection and attaching the subviews to our parent view.  But if the user pressed back and then chose another selection and transitioned to this view again, we would add further subviews to this parent view.  We need some way of keeping track of and disposing of our subviews.

To get round this problem, we have defined our own base class that uses underscore's extend method to *mixin* additional functionality with that of the Backbone.Collection class by adding custom methods onto the Backbone.Collections prototype.
{%gist 1368362 %}
On line 7, we are defining a method named **renderView** which takes a jquery wrapped DOM element, a string denoting a function of the JQuery object to call and a backbone view.  On line 8, we use javascript's ability to reference and call a method on an object like accessing a hashtable. This string will generally be something like 'append' or a means of attaching the subview to the DOM.  On line 9 we intialise an array to hold the subviews if it has not already been initailised using coffeescript's ruby like **||=** operator.  Finally we push the newly created view onto the array.

We call the method like this:
{% codeblock%}
  @collection.each (item, counter) ->
    self.renderView(self.el, 'append', new ReportBusinessUnitView(model: item, vent: self.vent))
{%endcodeblock%}
We pass the view's element, the string **'append'** which will be called on the jquery object in the renderView method like this **$.fn[func].call** and the subview we want attached.  This is very neat and I can take absolutely no credit for writing it.  We now have a store of all the views that are added.

All that remains is a means of disposing of these views and below are the two methods that take care of this on our custom BaseView class:
{%gist 1368467 %}
We can either call **dispose** on line 8   that will remove all subviews and the view itself or **disposeViews** on line 14 that will only remove the subviews.  Dispose takes care of not only unbinding any event handlers but also of removing the element from the DOM. We can simply call one of these methods in an appropriate place on the view like this:
{% codeblock%}
  @disposeViews()
{%endcodeblock%}
This has made backbone much more manageable although I cannot claim credit for much of the code.

That is it for this post and I will be amazed if anyone has made it this far.  Please feel free to leave any comments.  Below is the full listing of the BaseView class:
{%gist 1367986 %}