---
layout: post
title: "Ember.js - Client Side MVC is not Server Side MVC, Erase Your Brain"
date: 2013-03-23 15:05
comments: true
categories: JavaScript Ember
---
I'm writing this in the week ending 23 March 2013.  A week that future generations will refer to as hate on ember week.  A lot of people are complaining that ember has a barrier to entry and that barrier is the complexity or the amount of new information that the debutant has to take on board.  So I thought I'd pen a couple of posts that might help set the ground work for understanding ember.
###Client Side MVC is completely different than Server Side MVC###
I thought I would repeat the title for good measure.  I like a lot of front end developers (I am guessing) have ended up on the browser side of the divide by circumstance and partially because I enjoy coding on the client more.  It was not a choice I made.  I just worked on progressively more javascript heavy apps. 

I have worked with a number of server side mvc frameworks, and as it turns out, these server side incarnations are not actually true mvc in the same sense that the classic smalltalk mvc was.  I think they more accurately adhere to a <a href="http://en.wikipedia.org/wiki/Model2" target="_blank">model2</a> architecture.  Let us contrast the two.

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

The best way to flesh this out is with a practical code example:


 
