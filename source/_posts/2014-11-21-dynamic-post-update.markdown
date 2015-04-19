---
layout: post
title: "Ember.js - Programmatically Bind from a Handlebars helper"
date: 2014-11-21 10:44:46 +0000
comments: true
categories: JavaScript Ember
---
## Warning: The examples use Ember 1.7.1

This is a quick update to my last <a href="http://www.thesoftwaresimpleton.com/blog/2014/11/18/dynamic-content/">post</a> where I created a helper that bound data from an external json hash.

One problem I ran into was that if I was not creating links via the ```link-to``` helper then the properties were not bound.  In line 11 of the gist below, I am returning a simple string that will render the unbound property wrapped in a span tag.
{% codeblock bad.js %}
Ember.Handlebars.registerBoundHelper('getProperty', function(context, property, options) {
  var defaults, prop;
  defaults = {
    className: "",
    context: "this",
    avatar: false
  };
  property = Ember.merge(defaults, property);
  prop = context.get(property.binding);
  if (!property.hasOwnProperty("route")) {
    return new Handlebars.SafeString("<span>" + prop + "</span>");
  }
{% endcodeblock %}

The solution was to call out to the handlebars ```bind``` helper after updating the options hash on lines 2-5 below:
{% codeblock good.js %}
  if (!property.hasOwnProperty("route")) {
    options.contexts = [context];
    options.types = ["ID"];
    return Ember.Handlebars.helpers.bind.call(context, property.binding, options);
  }
{% endcodeblock %}
Here is an updated <a href="http://jsbin.com/fitale/27/edit" target="new">jsbin</a> with a full working example.

I am not sure if this is relevant for life after ```ember 1.7.1``` but these techniques have worked well for me thus far.