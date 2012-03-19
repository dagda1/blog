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
- The **addLead** method on line 12 takes a string of JSON as an argument, creates an Ember model object and uses the **setProperties** method to set multiple properties of the Ember model object with a hash which we create from the passed in string.
- On Line 15, the newly created object is added to the wrapped array with the **pushObject** method.  This reveals the beauty of the ArrayProxy, when pushObject is called, the bindings will automatically update and render the new lead in the view's output.  The **removeObject** method works the same in reverse.

##Problem 1 - Indexing the results ##
I will show the code for the handlebars template shortly, but first I want to show the first problem I came across and that was how to index the results of the table.  Below is an image that highlights the numbering of each row that I was after:
{%img /images/ember/results_index.png%}
Below is my handlebars template that will render each object added via the pushObject method. This is the template that the Ember view in the previous *gist* pointed to.
{% gist 2079992 %}
On line 13 of the gist, I am using the each helper which allows you to iterate over a list.  When I first came across the each helper used in conjunction with the ArrayProxy I thought it would re-render the html each time but this is not the case.  A new item is rendered via each for each new object inserted into the array via pushObject.  On line 13 I am binding the each helper to an accessible object instance named **Lead.leads_controller** which is actually an instance of the ArrayProxy defined in the previous gist.

When I first wrote this template, I used the **#collection** view helper instead of the **each** helper.  The **#collection** view helper creates a view for every item in the list.  I was informed again via twitter that the **#collection** view helper would soon be deprecated so I refactored to the each helper and created a  separate view for every item in the list which you can see on line 14.
{% gist 2096900 %}

{%codeblock%}
adjustedIndex: function(){
                 this.getPath('_parentView.contentIndex') + 1;
               }.property();
{%endcodeblock%}

