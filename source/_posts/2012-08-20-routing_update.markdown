---
layout: post
title: "Ember.js Routing - The Director's Cut"
date: 2012-08-20 17:32
comments: true
categories: JavaScript Ember
---
I've had a number of requests to update <a href="http://www.thesoftwaresimpleton.com/blog/2012/04/22/ember-js-routemanager/" target="_blank">this</a> post which discussed a solution to the lack of routing at that time of writing in <a href="http://emberjs.com/">ember</a>.  This solution used an addon called the <a href="https://github.com/ghempton/ember-routemanager" target="_blank">ember-routemanager</a> from <a href="https://twitter.com/ghempton" target="_blank">Gordon Hempton</a>.    

A lot in Ember has changed since that post and Ember now has its own full blown <a href="http://emberjs.com/guides/outlets/#toc_the-router" target="_blank">routing solution</a> that is not a million miles away from the route manager I blogged about.  I believe <a href="https://twitter.com/ghempton" target="_blank">Gordon Hempton</a> who created the origin ember-routemanager is now an ember core member.  

##Enough of the old and on with the Routing##
The ember router has been on the <a href="" target="_blank">master</a> branch for a while now and it is an emerging pattern that is prone to change and indeed has changed quite a bit in its short existence.  The ember routing solution is one of the main reasons I am drawn to ember and differs greatly from anything else out there in the **javascript mv*** space.

The Ember router extends the very elegant ember statemanager:
{%codeblock%}
Ember.Router = Ember.StateManager.extend({
});
{%endcodeblock%}
This is the third time I have blogged about ember/sproutcore statecharts/statemanagers/routers etc. (<a href="http://www.thesoftwaresimpleton.com/blog/2012/02/28/statemachine/" target="_blank">here</a> and <a href="http://www.thesoftwaresimpleton.com/blog/2012/04/22/ember-js-routemanager/" target="_blank">here</a>) and I really got excited the first time I came across this new pattern.  The statemachine pattern is a beautiful fit for routing it is surprising nobody else has come up with it as a solution until now.  

The basic premise is that you describe your application as a hierarchical tree of objects - one object per conceptual state like below: 
{%gist 3412899 %}

- On **line 1** we provide a subclass of the Ember.Router as the **Router** property of the application which in this instance is named **WZ**.  Naming the router subclass **Router** is a convention that you need to adhere to and I'll explain why later when I explain how an ember application gets initialised.
- **Line 2** tells the Ember.StateManager (which the Router derives from) to log state transitions to the console.
- **Line 3** sets the location property of the router to **hash** which means we will be using url fragment identifiers like #/post/1 for routing.
- On **line 4** a **root** state is provided. 


<!-- <a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
 -->
