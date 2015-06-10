---
layout: post
title: "Emberjs - Computed Properties and Promises.....don't"
date: 2015-06-10 16:06:35 +0100
comments: true
categories: Ember, JavaScript
---
A question I see coming up time and time again is how to return a resolved promise from a computed property.

There are a number of ways of doing this and I, like many have tried to solve this problem myself before realising that it is not a path you should go down.

I can flesh this out with an example that I originally solved using a computed property that resolved a promise.

I inherited a nasty bit of code on the server or API side of the project that I was working on that stored the relationship of different types af object in a generic postgres database column that used serialized yaml as the format to compound the uselessness.  This is particularly wrong for a number of reasons such as you can't query the sucker unless you use regular expressions.  It is completely unmaintainable and you should never, ever do this and Edgar F. Codd who gave us relational database theory would be turning in his grave.

An example where this was used is in a user's activity feed that contains audited events for a user of the application.  For example if the user or contact sent an email then their activity feed would contain an item like this:

{% codeblock %}
{
"activities": [{
    "id":210849,
    "tag":"user",
    "event":"sent_email",
    "time":"2015-06-06T17:01:51Z",
    "meta":{"email_id":607149}
}]
{% endcodeblock %}
The ```meta``` field on ```line 7``` comes from the serialized yaml column and is used as a generic bucket that can contain different types of relationships or other junk, making it extremely difficult to work with.  The answer to how to deal with this is refactor the server code and never, ever use serialized YAML but if you are stuck with this type of nonsense then how would you satisfy the ```email``` property of the template below?  We need to somehow or somewhere load the email instance from the ```meta.email_id``` property.
{% gist 6c7f4de4fce3250fa540 %}

When I first encountered this problem, my answer was to use a computed property that resolved a promise and then set itself:

{% codeblock cp.js %}
email: Ember.computed('activity.meta.email_id', function(){
  var store = this.get('store'),
      emailId = this.get('activity.meta.email_id'),
      self = this;

  if(!emailId) {
    return;
  }

  store.find('email', emailId).then(function(email) {
    self.set('email', email);
  }).catch(function(error) {
    //handle error
  });
})
{% endcodeblock %}
On ```line 11``` of the above, I set the property when the property resolves.

This will more or less work and here is a <a href="http://jsbin.com/rajori/4/edit?html,js,output" target="_blank">jsbin</a> that shows an example of how to do this.

There are a number of things wrong with this approach, namely we are trying to treat something that is inherently synchronous in an asynchronous manner.  The cp will fire a number of times due to the promise resolving and I have had a lot of pain with ember and observing dependent keys.  This can lead to the property referencing a partially resolved object or you can get cascading events on other cps and observers that reference a key firing in a sometimes recursive manner.  These are situations I now avoid like the plague and if you have any sense, you will heed my battle weary tone and avoid them too.

I've also seen computed properties that reference promise objects or have dependent keys on the state of the promise, but that is far too contrived to be credible and will lead to pain.  You will have partially resolved data and cascading events triggered.  Don't do this.

I develop with ember a lot different now than I did this time 12 months ago.

If you feel you need a computed property that will resolve a promise and you have read this, the answer is.......DON'T.

So what do we do?

Subscribing to data events throughout your application by using observers or waiting for computed property keys to update and trigger actions creates overhead but more importantly makes your application difficult to reason about and in my experience causes unreliable and inconsistent behaviour for reasons I have been writing about for a while now.

When data is passed from above rather than being subscribed to, we can react only when we know something has changed.  <a href="http://facebook.github.io/react/" target="_blank">React's</a> <a href="http://facebook.github.io/flux/docs/overview.html" target="_blank">flux</a> has popularised the top down approach and it makes nothing but sense.

So the answer is to let the resolved property be passed in from above in a top down fashion rather than *listening* for it to happen.  Ember is on board with this philosophy and it goes by the meme *data up and actions down* although I think this could do with some better fleshing out.

So the first refactoring is to create a service that will be charged with resolving the ```activity``` instance and taking care of unwrapping this horrible ```meta``` hash that can contain all kinds of junk.  I want this encapsulated and ready to use anywhere in my code.  With this in mind, I have created an ```ActivityService``` that is injected into all components through this ```initializer```:
{% codeblock initialize.js %}
Ember.Application.initializer({
  name: 'load-services',
  initialize: function(container, application) {
    var activityService = App.ActivityService.create({
      store: container.lookup('store:main')
    });
    application.register('activity-service:current', activityService, {
      instantiate: false
    });
    application.inject('component', 'ActivityService', 'activity-service:current');
  }
});
{% endcodeblock %}

A scaled down version of the ```ActivityService``` is below that takes care of loading the activity and unpacking this now infamous ```meta``` field:
{% codeblock activity_service.js %}
App.ActivityService = Ember.Service.extend({
  resolveActivity: function(activityId) {
    var self = this,
        store = this.store;

    return new Ember.RSVP.Promise(function(resolve, reject) {
      store.find('activity', activityId).then(function(result) {
        var activity = result.shallowCopy();

        store.find('email', result.get('meta.email_id')).then(function(result) {
          activity.email = result.shallowCopy();
          resolve(activity);
        });
      });
    });
  }
});
{% endcodeblock %}

The above code returns a promise that when resolved, will return a simple hash rather than the resolved ember-data instance.  I prefer not to bind or work directly with the ember-data instances and I discussed this the benefits of this in <a href="http://www.thesoftwaresimpleton.com/blog/2015/05/21/functional-ui-ember-1/" target="_blank">this post</a>.  I have created a simple ```shallowCopy``` extension to the ```DS.Model``` class that creates a shallow copy of the ember-data instance:
{% codeblock shallowcopy.js %}
DS.Model.reopen({
  shallowCopy: function(){
    var type = this.constructor,
        self = this,
        hash = {};

    type.eachAttribute(function(key, meta) {
      hash[key] = self.get(key);
    });

    return hash;
  }
});
{% endcodeblock %}

In the original example, I iterated over the user's activities like this:
{% gist 495c85bec1a0e8f8ffe2 %}

I will replace this with a component:
{% gist 026c34a98c53ca67c24d %}

The ```ActivityFeedComponent``` creates an array of activity hashes by calling out to the ```ActivityService``` and only pushing the resolved instances onto the ```activities``` array when the service has done what it needs to do:
{% codeblock ActivityFeedComponent.js %}
App.ActivityFeedComponent = Ember.Component.extend({
  activities: Ember.A(),
  _initialize: Ember.on('init', function(){
    var self = this;

    this.get('user.activities').forEach(function(a) {
      self.ActivityService.resolveActivity(a.get('id'))
      .then(function(activity){
        self.get('activities').addObject(activity);
      }).catch(function(error) {
        // handle error
      });
    });
  }),
  _tearDown: Ember.on('willDestroyElement', function(){
    this.get('activities').clear();
  })
});
{% endcodeblock %}

This will give me so many fewer problems than resolving the promise in a computed property as I did in the original example.  I only push onto the ```activities``` array when the service has done what it needs to do.  I am also working with copies of the ember-data instance which will curtail any unwanted side effects that I might encounter.  I will sleep better with this code.

The ```activity-feed``` component's template simply looks like this:
{% gist 27465d73363692d46fa7 %}

Here is a <a href="http://jsbin.com/juxeju/6/edit?html,js,output" target="_blank">jsbin</a> that brings it all together.

So my message is clear, don't resolve promises in your computed properties, nor should you have dependent keys on ```isFulfilled``` as I see some advocating.

You have been warned.
