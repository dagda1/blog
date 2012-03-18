---
layout: post
title: "Ember.js - Collection DataBinding"
date: 2012-03-18 15:40
comments: true
categories: JavaScript Ember
---
I've been hacking away at quite a complex results page that uses <a href="http://emberjs.com/">Ember.js</a> and <a href="http://www.handlebarsjs.com">Handlebars</a> templating to bind json objects that are returned from a websocket connection on the server.  This proved quite complex and I thought I would blog about it in order to help anybody else who struggles with what I am guessing are some common problems.  I am also hoping that somebody with greater Ember knowledge than mine can point me to a better way or correct any of my misconceptions.
##Scenario##
Below is a screen capture of what I finally ended up with.  
{%img /images/ember/results.png%}
**Warning:** I have used nested tables to display these results, please leave a comment suggesting a better way if this offends!

Each row in the table displays the company details from a website screen scraping search operation.  Each row can be expanded to display any additional child detail.  The child detail can contain any additional email addresses and contact details that have been scraped from a web site search and belong to the parent row.

##Enter the Ember.ArrayProx##
One of Ember's strengths is its excellent databinding support and out of the box, it comes with the <a target="_blank" href="http://ember-docs.herokuapp.com/symbols/Ember.ArrayProxy.html">Ember.ArrayProxy</a>.  The ArrayProxy is a construct that wraps a native array and adds additional functionality for the view layer.  In this example, I am going to use the **Ember.ArrayProxy** as a means to publish a collection of objects so that I can easily bind the collection using the handlebars **#each helper** (more on this later). 

It is worth noting that you might also see the **ArrayProxy** in the wild named as the **ArrayController**.  I was informed by somebody on twitter that the ArrayController is just an alias to ArrayProxy.  The ArrayController is a throwback to Ember's sproutcore days and will at some stage be deprecated.  I prefer ArrayProxy because it is not a controller in the MVC sense of the word and **ArrayProxy** is a better description of the object's behaviour.

Below is how my extended ArrayProxy ended up:
{% gist 2078762 %}

- The constructor for the ArrayProxy is defined on **line 2**.
- On line 3 I am calling the constructor of the superclass ArrayProxy with **@_super()**.  If you do not do this the ArrayProxy will not initialise correctly.  I only found this out after a lot of head scratching as to why my ArrayProxy was not working.
- On line 5, we are setting the initial array that will be wrapped by the ArrayProxy.  This is done by using the **content** property. The **content** property must be an object that implements either **Ember.Array** or **Ember.MutableArray**.  For the starting position of our view, we want the ArrayProxy bound to an empty array.  **Ember.A()** will return an empty Ember array.
- On line 7, we create a view that we will attach to the DOM.  The virtual path to the handlebars template is set on line 8 and on line 10 we attach the initial view to the DOM.

