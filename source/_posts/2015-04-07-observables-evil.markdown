---
layout: post
title: "Emberjs - Observers the Root of all Evil"
date: 2015-04-07 17:13:44 +0100
comments: true
categories: Ember, JavaScript
---
### Observers Are Like Crack Cocaine only more destructive....
After 2+ years of working on the same emberjs application, I have learned the hard way that the observable primitive that ember exposes is something that you and your team should outlaw this very instant.  One problem the ember community has is that a lot of the code examples that you might find online use observers.  If you are new to ember then you would have no reason not to trust these well meaning examples.

Below is the first one I plucked from the multitude:
{% codeblock observer.coffee %}
App.DynamicBoundTextField = Ember.TextField.extend
  placeholderBinding: 'content.name'
  # Update the remote property's value when the value of
  # our text field changes
  _setRemoteValue:(->
    val = @get('value')
    @set("controller.data.#{@get('content.key')}", val) if val?
  ).observes('value')
{% endcodeblock %}
This post is not a dig at the person who wrote this as I have gone down this horrible route many times.  I am probably more guilty than most.  I've also used observers in more contrived examples than this.  This is definitely a case of let ye without guilt cast the first stone.

The above looks very reasonable and I don't have to write any code to keep things in sync, I pass that responsibility to ember.  For me this is where the problems start.  I've lost control of the process, what if I want to <a href="https://remysharp.com/2010/07/21/throttling-function-calls" target="_blank">throttle or debounce</a> the user input?  If you have written any sort of user input control then you will know that throttling is a technique to limit the amount of times a function is called in response to user input.  If I am using the above code then how do I throttle the user's input?  You could definitely cobble together some sort of hack but you have now lost control and you are making the solution fit the problem.


I recently looked at a bug to do with a component that uses the html5 <a href="http://html5doctor.com/the-contenteditable-attribute/" target="_blank">contenteditable</a> attribute to expose some very, very simple WYSIWYG editing to the user.  The component did something similar by observing the value of the contenteditable ```div``` and updating a value whenever the content changed.  Working with contenteditable can be quite fiddly and if you are changing the markup, you often have to reposition the cursor at the end of the div's content.  The bug I was fixing was that the cursor did not always reposition itself at the right time after the user had finished their input and the ember run loop had flushed the changes.  I found the code very difficult to reason about but more importantly it was outside of my control.

I removed the observer and refactored to this:
{% codeblock input.js %}
input: function(e) {
  var route, text;
  text = this.$().text();
  this.set('prop', text);
  route = this.routeLink(text);
  this.$().html(route);
  this.setEndOfContentEditble();
}
{% endcodeblock %}

I am using the <a href="https://developer.mozilla.org/en-US/docs/Web/Events/input">input</a> event to do the simple things and update properties manually.  I have control of what is going on.

I removed all observers and all computed properties and sanity was instantly returned.

The above is a ridiculously simple example that serves to illustrate how difficult coding can be in a really simple situation when you have lost control of what is going on.  With observers and computed properties, it is down to ember, the run loop and a whole host of other contributing factors to decide when updates happen and observers fire.  You really do not need the headaches that this can cause.  This can get worse if you have computed properties and observers watching the dependant keys as chain reactions and other unexpected events can occurr.  This is the antithesis of side effect free coding.  This is the wild, wild west.  Ember seems to be in the process of removing a lot of complexity and I think they should add observers and computed properties to the list.  The more I use ember, the more I just want to use POJO objects and arrays.  My worry for ember is that it went to the trouble of creating its own object model that starts at the very root ancestor.  There is a lot of complexity inherint in such an approach.

Below are some of the problems and pain that can be caused by illicit use of observers and computed properties.

- it is difficult to hold a mental model of the code if observers and computed properties are used to orchestrate data flow.
- You are pushing the responsibility of when things happen to ember and outside of your control.
- Observers and computed properties drive state/data flow into unexpected and mentally troubling paths.
- Observers have no context, you have no idea what triggered the change.
- Observers can cause chain reactions of other observers and can fire multiple times.
- Mutating state changes in response to an observer firing breaks the law of demeter.

Observers and computed properties are seemingly convenient but they bring large scale trouble into simple situtations.

I would like to see a reflux like data flow story in ember and I blogged more about this <a href="http://www.thesoftwaresimpleton.com/blog/2015/02/12/emberjs-data-down/" target="_blank">here</a> and <a href="http://www.thesoftwaresimpleton.com/blog/2015/03/13/ember-reflux/" target="_blank">here</a>.

I've also recently started this <a href="https://github.com/dagda1/ember-flow" target="_blank">project</a> which I will flesh out more before blogging about it.

The first path from addiction to these ugly primitives is to admit you have a problem.  After that reach out and find the nearest support group in your neighbourhood.

Remember you are not alone.
