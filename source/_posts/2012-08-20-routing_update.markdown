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
This is the third time I have blogged about ember/sproutcore statecharts/statemanagers/routers etc. (<a href="http://www.thesoftwaresimpleton.com/blog/2012/02/28/statemachine/" target="_blank">here</a> and <a href="http://www.thesoftwaresimpleton.com/blog/2012/04/22/ember-js-routemanager/" target="_blank">here</a>) and I really got excited the first time I came across this new pattern.  The statemachine pattern is a beautiful fit for routing and it is surprising nobody else has come up with it as a solution until now.  

The basic premise is that you describe your application as a hierarchical tree of objects - one object per conceptual state like below: 
{%gist 3412899 %}

- On **line 1** we provide a subclass of the **Ember.Router** as the **Router** property of the application which in this instance is named **WZ**.  Naming the router subclass **Router** is a convention that you need to adhere to and I'll explain why later when I explain how an ember application gets initialised under the new regime.
- **Line 2** tells the Ember.Router to log state transitions to the console.
- **Line 3** sets the location property of the **Router** to **hash** which means we will be using url fragment identifiers like **#/post/1** for routing.
- On **line 4** a **root** state is provided. As the name **root** suggests, all other routes (or states as I still think of them) will either be direct children of the **root** route or grandchildren of the **root** route in true composite fashion.  There can only one root route or root state.  The **root** route is acts as the container for the set of routable states **but** is not routable itself.
- On **line 5** and **line 8** are two such child routes or states of the **root** route named **index** and **home** respectively.  
- Every child route of the **root** route can have an optional **route** property describing the URL pattern you would like to detect.  On **line 6**, the **index** route has a route property with a value of <strong>'/'</strong> which will correspond to the home page of the application and on **line 9**, the home route has a route property of **/home**.
- On **line 10** is the **connectOutlets** method that you can override in each child route that provide a mechanism for changing what view hierarchy is rendered onto the page as the application state and url changes.  More on this later.

When an Ember application loads, Ember will parse the URL and attempt to find and Ember.Route within the Router's child route hierarchy that matches the url.   Loading the page at '/' will first of all transition transition the **router** to the first route name **root** and then then to the subsate or child route where it can find a match on the url.  In this case it will find a match on the route property of the **index** route and will transition to that route.  The **index** route simply redirects to the **home** route.

Loading the page at '/' will detect a route property of **root.index.home**.
<!-- 
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
