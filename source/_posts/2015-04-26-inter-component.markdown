---
layout: post
title: "emberjs - Communication Between Components"
date: 2015-04-26 07:01:03 +0100
comments: true
categories: Ember, JavaScript
---
###Communication Between Components

A question that I see frequently asked is how can a parent component communicate with child components.

A child component can communicate with a parent in a number of ways with the most commmon at this time of writing with ```ember 1.11.1``` is to use <a href="http://guides.emberjs.com/v1.10.0/components/sending-actions-from-components-to-your-application/" target="_blank">sendAction</a>.

For example, a parent component can tell a child component what its primary action is or give a named action to its children:
{% gist 8904db18b347eb337fca %}

And the child component can then call this action:
{% codeblock child1.js %}
App.XPersonComponent = Ember.Component.extend({
  actions: {
    callParent: function(person) {
      this.sendAction("action", person);
    }
  },
{% endcodeblock %}

I have never warmed to the abstraction of callable actions existing in an ```actions``` hash and from what I can tell, it will be possible in ```ember 2.0``` to simply pass actions around as plain old javascript functions which is something that I welcome.

In the mean time, it is possible to simulate this behaviour by creating a handlebars bound helper that returns a higher order function:

{% codeblock makeAction.js %}
Ember.Handlebars.registerBoundHelper('makeAction', function(){
  var actionName = arguments[0],
      args = Array.prototype.slice.call(arguments, 1, -1),
      self = this;

  args.unshift(actionName);

  return function() {
    return self.send.apply(self, args);
  };
});
{% endcodeblock %}

A parent component can then yield this to a child component:

{% gist 2c5aac5208128238125d %}

Here is a <a href="http://emberjs.jsbin.com/vivasa/16/edit" target="_blank">jsbin</a> that shows this in action.

###Parent Component calling Children
A child component calling a parent is fairly straight forward and is well laid out but what about when a parent component wants to call its children?

What do we do if we have a parent that wants its chidren to do something like to zoom to an element or something that is not a reaction to some state mutation?  The official ember guidance is you should use the **actions up and data** down paradigm where you push data changes down to the child components but there are occasions when this does not fit.  There are times when you want a child component to do something when no data has changed.  You can certainly fudge this but mutating a state change to get a child component to react is pretty awful in my opinion and not something that you should do.  This is an occassion when actions up and data down does not go far enough, which I have previously blogged about <a href="http://www.thesoftwaresimpleton.com/blog/2015/02/12/emberjs-data-down/" target="_blank">here</a> and <a href="http://www.thesoftwaresimpleton.com/blog/2015/03/13/ember-reflux/" target="_blank">here</a>.

So what do we do?

####Subscribe to Parent Events
The first way I solved this was to use **Ember.Evented**.  A component can mix in the  <a href="http://emberjs.com/api/classes/Ember.Evented.html" target="_blank">Ember.Evented</a> mixin:
{% codeblock people.js%}
App.XPeopleComponent = Ember.Component.extend(Ember.Evented, {
  actions: {
    callChildren: function() {
      this.trigger('parentCall');
    },

    receiveFromChild: function(person) {
      alert("received " + person.get('name'));
    }
{% endcodeblock %}

**Ember.Evented** is mixed into the component and ```trigger``` is called on ```line 3``` of the above code sample to broadcast an event to any listening children.

Below is how a child component can subscribe to these events:
{% codeblock child2.js %}
App.XPersonComponent = Ember.Component.extend({
  _setup: Ember.on('didInsertElement', function(){
    this.get('notifyer').on('parentCall', this, 'onParentCall');
  }),

  onParentCall: function() {
    alert('parent called with ' + this.get('person.name'));
  }
});
{% endcodeblock %}

```Line 3``` subscribes to the event and supplies a handler which will be called when the parent triggers the event.

I don't have a great big problem with this although it is not that nice having to explicitly subscribe via the parent component as opposed to just subscribing to an event.

Here is a <a href="http://emberjs.jsbin.com/vivasa/16/edit" target="_blank">jsbin</a> that shows this in action.

####Registration with the parent component
Another way of solving this is for the child component to register itself with the parent:
{% codeblock register.js %}

App.XPersonComponent = Ember.Component.extend({
  _setup: Ember.on('didInsertElement', function(){
     var parent = this.nearestOfType(App.XPeopleComponent);

     parent.registerChild(this);
  }),

  parentCalling: function() {
    alert('parent called with ' + this.get('person.name'));
  }
});
{% endcodeblock %}

The child component finds the parent on ```line 3``` of the above and below is the ```registerChild``` method that is used to register the child component:
{% codeblock parent2.js %}
App.XPeopleComponent = Ember.Component.extend( {
  children: Ember.A(),

  registerChild: function(child) {
    this.children.pushObject(child);
  }
});
{% endcodeblock %}

I think this is less favourable because of the very tight coupling.  <a href="http://emberjs.jsbin.com/vivasa/17/edit" target="_blank">Here</a> is a working jsbin that shows this in action.

####Register via Binding
Sam Selikoff offered <a href="http://www.samselikoff.com/blog/getting-ember-components-to-respond-to-actions/" target="_blank">this</a> way of registering child components via a binding.

First, add a ```_register``` method to the component that executes on init:
{% codeblock parent3.js %}
App.FullscreenMapComponent = Ember.Component.extend({
  _register: function() {
    this.set('register-as', this); // register-as is a new property
  }.on('init')

});
{% endcodeblock %}

Then, when rendering the component, supply a property to bind ```register-as``` to:

{% gist 8bb8a57bb19898754253 %}

Now, the controller has a reference to the component, and can call methods directly on it:
{% codeblock route.js %}
App.PinsRoute = Em.Route.extend({
  actions: {
    focusSelectedPin: function() {
      this.controller.get('fullscreenMap').panToSelectedPin();
    }
  }

});
{% endcodeblock %}

I think this method suffers from the same coupling as the other methods.

So what I want is a way that is less coupled.  I want the publisher and the subscriber to know as little about each other as possible.  I also do not want to use state mutation as a means of communication.

**UPDATE:**  Somebody mentioned in the comments that a global event bus is anohter way of achieving what is required and I created this <a href="http://www.thesoftwaresimpleton.com/blog/2015/04/27/event-bus/" target="_blank">post</a> that explains how to achieve this.

####clojure's core.async
I love this quote from the first sentence of <a href="http://clojure.com/blog/2013/06/28/clojure-core-async-channels.html" target="_blank">this</a> post on the clojure blog about core.async.

{% blockquote %}
There comes a time in all good programs when components or subsystems must stop communicating directly with one another.
{% endblockquote %}
The roots of ```core.async``` go back to <a href="http://en.wikipedia.org/wiki/Communicating_sequential_processes">Hoare's Communicating Sequential Processes (CSP)</a>.

I've done a good bit of hobby development with clojurescript and I love using core.async channels for communication between components.  It can be used synchronously and asynchronously.

With core.async, you create channels which can be thought of as independent threads of activity (I don't mean threads as in multi-threaded) that can be published to or subscribed to.

I have previously blogged about <a href="http://www.thesoftwaresimpleton.com/blog/2014/12/30/core-dot-async-dot-mouse-dot-down/" target="_blank">Handling Mouse Events With core.async</a>.

Core.async has nailed the decoupling of publishers and subscriers via queueable channels.
###Epilogue
With ```core.async``` the publisher and subscriber know nothing about each other which is really the goal of event based systems and I think both ember and react suffer from the model of handing down functions from parent to child as there is a high degree of coupling.

It would be interesting to see how ember was if we were dealing with channels as a means of communication rather than passing down callable functions.

There is a <a href="https://github.com/ubolonton/js-csp" target="_blank">js-csp</a> library and <a href="https://twitter.com/swannodette" target="_blank">David Nolen</a> wrote this post about <a href="http://swannodette.github.io/2013/08/24/es6-generators-and-csp/" target="_blank">ES6 Generators Deliver Go Style Concurrency</a>.

Ember is going with the data down, actions up meme but there are times when this does not fit and activating state changes as a means of communication can only lead to trouble.
