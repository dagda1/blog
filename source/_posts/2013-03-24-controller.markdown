---
layout: post
title: "The Ember Controller - Same same, but different"
date: 2013-03-24 19:00
comments: true
categories: JavaScript Ember
---
I have travelled around Vietnam twice and they have a the following very apt saying:
{%blockquote %}
Same same but different
{% endblockquote %}
I think this saying is descended from the ember.js controller.  The ember.js controller is similar in many respects to a controller object that you might find in a server side framework like rails but at the same time it is quite different.  Let us flesh this out.

In my last <a href="http://www.thesoftwaresimpleton.com/blog/2013/03/23/client-side-mvc/">post</a> I made this statement:
{%blockquote%}
a model can notify the view of any changes via the observer pattern.
{%endblockquote%}
As it turns out, the above statement is not entirely true or at least, I purposely omitted some facts and not all the actors of the piece received a mention.  A model does indeed notify the view of any changes but it does so by way of a middle man who sits in between the view and the model.  This middleman is of course the ember.js <a href="http://emberjs.com/guides/controllers/">controller</a>.  The ember api docs portray this shadowy middleman as this:
{%blockquote %}
In Ember.js, controllers allow you to decorate your models with display logic. In general, your models will have properties that are saved to the server, while controllers will have properties that your app does not need to save to the server.
{%endblockquote%}
As always the best way to illustrate this is with some code


