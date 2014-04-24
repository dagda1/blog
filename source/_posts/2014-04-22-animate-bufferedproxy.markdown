---
layout: post
title: "ember.js - Animating Deletes with the BufferedProxy"
date: 2014-04-22 21:12:33 +0100
comments: true
categories:  JavaScript Ember
---
I encountered an interesting scenario in my day job that initially had me scratching my head for a way to solve and seems worthy of a post.  The real world application that I am working on actually has real world todos!  The application lists the todos as is illustrated in the screenshot below and they can be completed in the time honoured fashion of clicking the checkmark or tick on the left hand side of the todos:
<br/>{%img /images/todos.png%}

One of the requirements for this list is to display only the todos that have not been completed.  Ember does make this ridiculously easy and we could even bind our list to the new <a href="https://github.com/emberjs/ember.js/blob/v1.5.0/packages/ember-runtime/lib/mixins/enumerable.js#L384" target="_blank">filterBy</a> computed property like this:
{% gist 11193486 %} Here is a working <a href="http://jsbin.com/lomix/2/edit" target="_blank">jsbin</a> that shows how easy this is.

The problem with the approach outlined in the <a href="http://jsbin.com/lomix/2/edit" target="_blank">jsbin</a> is that, when the checkbox is checked and the bindings are flushed, the view that has been rendered for that particular todo gets instantly destroyed.  There are currently no hooks that allow for animations in ember and this is particularly true when it comes to removing items from a bound list or indeed destroying views.  This leaves you as the developer to resort to some sort of trickery.  I would personally like to see an extra runloop queue or a view lifecycle event that returned a promise.  Only when the promise has been resolved would the view's destroy method be called.  There was some discussion about this before ember 1.0 was released but we are heading towards ember 1.6 and I think we need to discuss this again.

My requirements for this todo list are even more convulted because I want to visually indicate that the todo has been checked and also give the user a 4 second chance to change their mind and uncheck the todo before the todo disappears from the list.  As you can see from the jsbin, the moment the checkbox is checked, the todo just vanishes instantly.  So, what can be done?

###The BufferedProxy
The application that I am working on has been in active development for over a year and we still use version 0.14 of ember-data which was the last version of ember-data that was released before the much publicised reboot.  I have made a solemn oath to my client that I would not even think of upgrading until ember-data reached 1.0 after the pain we suffered with other ember-data upgrades. One problem with this version of ember-data (I cannot speak for the latest version) is that it is terribly annoying to work with models that are left in **isDirty** or invalid states. I am not going to go into detail here but the state machine assertions are a source of depression for anybody that has had the experience.  I have since patched up my slightly bastardised version of ember-data but what makes life easier is to actually not bind to an ember-data model and only create the model when you are absolutely sure that your model is ready to be persisted.

The <a href="http://coryforsyth.com/2013/06/27/ember-buffered-proxy-and-method-missing/">BufferedProxy</a> has been an absoute god send for my productivity with this version of ember-data.  The **BufferedProProxy** is a mixin that I mixin into controllers that uses ember's lesser know **method_missing** like feature (checkout **unknownProperty** and **setUnknownProperty**) to store changes to a controllers model in a buffer that you can either set explicity or you can cancel them completely without the model ever being changed.  This is perfect for ember-data models.  We can only apply the modifications if we are happy that the model is valid or that the user has not navigated away, cancelled, left the building, been set on fire, shot, kidnapped etc.

The **BufferedProxy** is ideal for my requirements which are:

1. I want to delay setting **isFinished** on the model so that the user has a few seconds to change their mind and undo the change.
2. I want to animate the deletion of the todo view to make it look a bit more polished than it is in the original bin where it instantly disappears.

Here is a a working <a href="http://jsbin.com/gufil/9/edit" target="_blank">jsbin</a> of what I ended up with.  There is a delay in persisting the change that gives the user an opportunity to cancel and the delete has a basic animation.  If you re-check a todo, then it will not be removed or if you leave it checked, a basic animation will kick in and the todo will be removed when the animation is completed.

The first thing to notice is that I am using **render** instead of a component to render my todos which gives me a clean separation between controller and view which I think is better for this situation because I want to **mix in** the **BufferedProxy** into the controller and not the component even though I am pretty sure it would work with a component:
{% gist 11209236 %}

In the **TodoController**, I observe the **isFinished** property for changes in the following handler:
{% gist 11209443 %}

- On **line 4** I am checking if the todo **isFinished** and the **hasBufferedChanges** property lets me know that there are changes in the **BufferedProxy** that are ready to be applied.
- On **line 5**, I create an inline function that will be called after a delay.  The delay is to give the user the opportunity to change their mind and cancel the change.
- The inline function on **line 5** will raise an event on **line 6** to the **TodoView** that is listening for these events and will give the view the opportunity to animate the delete.  The timer is cancelled on **line 7**.
- On **line 14** we handle the case where the user has changed their mind.  We clear the timer and then persist the change if the **BufferedProxy** has changes.

Below is the **TodoView** that handles the **animateFinish** event that is triggered by the **TodoController** on **line 6** of the above gist.
{% gist 11209896 %}

- **line 5** sets up the event handler for **animateFinsh** events that are raised by the **TodoController**.
- **lines 10 and 11** uses jQuery's **fadeOut** for some basic animation which calls a controller action to persist the changes when the animation has finished.

All that remains is to show the **completeFinish** action that is called in the above gists:
{% gist 11210132 %}

- On **line 4** of the above gist, I apply the changes from the **BufferedProxy** by calling the **applyBufferedChanges** method which will transfer the changes from the buffer to the model.
- On **line 6**, I persist the model.

I hope if nothing else that this post has highlighted the complexities of adding animations to ember and especially in the destroy phase.  Animation is a must have for modern day javascript single page applications and not a nice to have.  I'm not sure if this is on the ember roadmap but I think this is something that can no longer be ignored.  I would like to see a run loop queue or view lifecycle event that allowed me to return a promise with the view's destroy method only being called when that promise has resolved.

Please comment if any of the above has unsettled you.
