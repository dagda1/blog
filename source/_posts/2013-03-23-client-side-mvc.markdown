---
layout: post
title: "Ember.js - Client Side MVC is not Server Side MVC, Erase Your Brain"
date: 2013-03-23 15:05
comments: true
categories: JavaScript Ember
---
{%img /images/shocktreatment.png %}

I'm writing this in the week ending 23 March 2013.  A week that future generations will refer to as hate on ember week.  A lot of people are complaining that ember has a barrier to entry and that barrier is the complexity or the amount of new information that the debutant has to take on board.  So I thought I'd pen a couple of posts that might help set the ground work for understanding ember.

####I think it is very important to grasp this concept before moving on to any of the other objects in ember such as the router, controller, ItemController etc., etc..####

###Client Side MVC is completely different than Server Side MVC###
I thought I would repeat the title for good measure.  I have ended up being a mainly front end only developer by circumstance and partially because I enjoy coding on the client more.  It was not a choice I made.  I just worked on progressively more javascript heavy apps. 

I have worked with a number of server side mvc frameworks, and as it turns out, these server side incarnations are not actually true mvc in the same sense that the classic smalltalk mvc was.  I think they more accurately adhere to a <a href="http://en.wikipedia.org/wiki/Model2" target="_blank">model2</a> architecture.  I had to stop thinking in terms of what I knew to grasp the new paradigms.  Let us contrast the two.

A typical http request/response of a server side mvc framework is this:

- There is a single point of entry into the system.  All requests are going to come into the system by the browser hitting a route.
- The web server processes determines which route it belongs to and dispatches that request to the corresponding controller action.
- The controller will then retrieve the appropriate model before handing the model or some wrapped viewmodel to the view to do the rendering.
- Bottom line is, each GET request usually results in a big blob of something being returned.  That something could be html, json, xml, text or whatever.

The main difference with the client is that with a server side framework, the objects only exist for the length the http request.  Http is a stateless protocol and this constrains the real mvc pattern that was popularised by smalltalk.  The views on the server can only receive a request, and dump out some data.

On the client side, there are no such constraints as the objects can live as long as the browser session.  What finally triggered off all the right mental associations about client mvc for me is the following statement:
{%blockquote%}
a model can notify the view of any changes to itself via the observer pattern.
{%endblockquote%}
The inverse of the previous statement is also true in that a view can notify the model of any changes via the observer pattern.

The best way to flesh this out is with a practical code example.  Below is the possibly the least amount of code I could create to illustrate how a view is updated with changes to the model and vice versa.  You can view this code in the following <a href="http://jsfiddle.net/dagda1/EhyMR/4/" target="_blankd">jsfiddle</a>.

First up we have the code that will create a very simple object that we can use to i

Next we have the markup which is an ember flavour of the very powerful client side templating language <a href="http://handlebarsjs.com/" target="_blank">handlebars</a>:
{{%gist 5231890 %}}



 
