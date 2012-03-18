---
layout: post
title: "Ember.js - Collection DataBinding"
date: 2012-03-18 15:40
comments: true
categories: JavaScript Ember
---
I've been hacking away at quite a complex results form which uses <a href="http://emberjs.com/">Ember.js</a> and <a href="http://www.handlebarsjs.com">Handlebars</a> templating to bind json objects that are returned from a websocket connection on the server.  This proved quite complex and I thought I would blog about it to help anybody else who struggles with what I am guessing are some common problems.  I am also hoping that somebody with greater Ember knowledge than mine can point me to a better way or correct any of my misconceptions.
##Scenario##
Below is a screen capture of what I finally ended up with.  
{%img /images/ember/results.png%}
**Warning:** I have used nested tables to display these results, please leave a comment suggesting a better way if you disagree with this approach.

Each row in the table displays the company details of a website screen scraping search operation.  Each row can be expanded to display any additional child detail.  The child detail can contain any additional email addresses that have been found and any additional contact details that have been scraped from a web site search.

I have been using Backbone.js in my day job for a number of months now and I tend to view things in a Backbone.js kind of way. With Backbone.js, everything fits into Model, Collection or View. It is a little to easy to end up with fat views that do way to much.

##Enter the Ember.ArrayProx##
One of Ember's strengths is its excellent databinding support and out of the box, it comes with the <a target="_blank" href="http://ember-docs.herokuapp.com/symbols/Ember.ArrayProxy.html">Ember.ArrayProxy</a>.  As the name suggests, the ArrayProxy is a wrapper around an array or in this case an Ember.Array that forwards all requests and provides a number of useful databinding capabilities.  In this example, I am going to use the **Ember.ArrayProxy** as a means to publish a collection of objects so that I can easily bind the collection using the handlebars **#each helper** (more on this later). 