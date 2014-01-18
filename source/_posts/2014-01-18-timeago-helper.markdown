---
layout: post
title: "emberjs - creating a timeago helper"
date: 2014-01-18 13:24:18 +0000
comments: true
categories:  JavaScript Ember
---
Freetime is very hard to come by when you have two small children under three but I actually had a couple of hours spare at this time of writing and  had set myself the challenge of implementing an <a target="_blank" href="http://emberjs.com/">ember.js</a> <a href="http://timeago.yarp.com/" target="_blank">timeago</a> handlebars helper when I came across this <a href="https://github.com/jgwhite/ember-time" target="_blank">example</a> by <a href="https://twitter.com/jgwhite" target="_blank">@jgwhite</a> that does exactly that.  The <a href="https://github.com/jgwhite/ember-time/blob/master/README.md" target="_blank">README</a> gives a very good breakdown of how to implement the helper.

I've made some slight changes which you can see in this working <a href="http://jsbin.com/iWUKAKiH/4/edit" target="_blank">jsbin</a>.  My view is pretty much the same apart from that I am comparing raw javascript dates instead of using <a href="http://momentjs.com/" target="_blank">moment.js</a>.

The key to making this work is the <a href="http://emberjs.com/api/classes/Ember.Observable.html#method_notifyPropertyChange" target="_blank">notifyPropertyChange</a> method which is a convenience method that calls <a href="http://emberjs.com/api/classes/Ember.Observable.html#method_propertyWillChange" target="_blank">propertyWillChange</a> and <a href="http://emberjs.com/api/classes/Ember.Observable.html#method_propertyDidChange" target="_blank">propertyDidChange</a>.  Calling both these methods consecutively is actually what happens behind the scenes when you call **set()** on an object and the combination triggers a notification to all registered obervers that the property has changed.

My handlebars helper is ever so slightly different and possibly worth a brief comment and is listed below:
{% gist 8490904 %}

**Line 4** adds a new property to the the hash property of the options argument.  Handlebars will mix any property in this options hash into the view. You can also set up bindings and other goodness for each property that you add by using **option.hashTypes** hash.  I am not doing that in this instance but it is worth noting that you can.

**Line 6** uses the <a href="http://emberjs.com/api/classes/Ember.Handlebars.helpers.html#method_view" target="_blank">view helper</a> to insert a new instance of the **App.TimeAgoView** into the output.  It is worth noting that this is the helper that is called by handelbars for normal view declarations like this:
{% codeblock %}
&#123;&#123;view App.SomeView&#125;&#125;
{% endcodeblock %}  

The upshot of all this is that I feel a bit deflated as I was really looking forward to sinking my teeth into this.  I guess I have <a href="https://twitter.com/jgwhite" target="_blank">@jgwhite</a> to thank for that. 

I now need a new chanllenge.......and something that I have actually written to blog about.....