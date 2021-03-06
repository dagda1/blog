---
layout: post
title: "ember.js - Computed Property for all keys of an object"
date: 2014-03-11 22:14:01 +0000
comments: true
categories:  JavaScript Ember
---
I came across a situation today where I wanted to create a computed property which would recalculate itself every time one of the properties of an object changed.  The lazy and tedious thing to do would have been to simply list them out like this:
{% codeblock long.js %}
previewHeaders: Ember.computed('headerInfo.firstName', 'headerInfo.surname', 'headerInfo.email', function(){
  //do stuff
})
{% endcodeblock %}

This is tedious for a number of reasons but the main one is that I have to remember to update the dependent key list, every time the object changes.

Below is the solution I came up with:
{% codeblock just.js %}
App.computed.addAllKeysProperty = function(context, propertyName, objectPath, func) {
  var args, subject;
  subject = context.get(objectPath);
 
  args = Ember.keys(subject).map(function(key) {
    return "" + objectPath + "." + key;
  });

  args.push(func);

  return Ember.defineProperty(context, propertyName, Ember.computed.apply(this, args));
};
{% endcodeblock %}

This is slightly more convoluted that I would have liked as I could not create a real property because I need the **context** argument to be an instance on **line 3**  so that I can call **Ember.keys** on this instance. Instead of creating an actual property, I use **defineProperty** on **line 11** to create the property at runtime.  I create the dependant key list by using **Ember.keys** to iterate over the subject's properties and then call **Ember.computed.apply** to equal the call from the gist at the top of the post.

Below is the calling code:
{% codeblock init.js %}
init: function() {
  this._super.apply(this, arguments);
  return App.computed.addAllKeysProperty(this, 'previewHeaders', 'headerInfo', function() {
    //do stuff
  });
},
{% endcodeblock %}

It is slightly dissatisfying in that I could not create an actual property and if you can suggest a better way then please leave a comment below.

