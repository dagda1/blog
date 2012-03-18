---
layout: post
title: "Ember.js - Collection DataBinding"
date: 2012-03-18 15:40
comments: true
categories: JavaScript Ember
---
I've been hacking away at quite a complex results form which uses <a href="http://emberjs.com/">Ember.js</a> and <a href="http://www.handlebarsjs.com">Handlebars</a> templating to bind json objects that are returned from a websocket connection on the server.  This proved quite complex and I thought I would blog about it to help anybody else who struggles with what I am guessing are some common problems.
##Scenario##
Below is a screen capture of what I finally ended up with.  
{%img /images/ember/results.png%}
**Warning:** I have used nested tables to display these results, please leave a comment suggesting a better way if you disagree with this approach.

Each row in the table displays the company details of a website screen scraping search operation.  Each row can be expanded to display the child detail.  The child detail can contain any additional email addresses that have been found and any additional contact details that have been scraped from a web site search.