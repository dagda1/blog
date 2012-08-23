---
layout: post
title: "Ember.js Routing - The Director's Cut"
date: 2012-08-20 17:32
comments: true
categories: JavaScript Ember
---
I've had a number of requests to update <a href="http://www.thesoftwaresimpleton.com/blog/2012/04/22/ember-js-routemanager/" target="_blank">this</a> post which discussed a solution to the lack of routing at that time of writing in <a href="http://emberjs.com/">ember</a>.  This solution used an addon called the <a href="https://github.com/ghempton/ember-routemanager" target="_blank">ember-routemanager</a> from <a href="https://twitter.com/ghempton" target="_blank">Gordon Hempton</a>.    

A lot in Ember has changed since that post and Ember now has its own full blown <a href="http://emberjs.com/guides/outlets/#toc_the-router" target="_blank">routing solution</a> that is not a million miles away from the route manager I blogged about.  I believe <a href="https://twitter.com/ghempton" target="_blank">Gordon Hempton</a> who created the origin ember-routemanager is now an ember core member which might explain the start similarities.  

##Enough of the old and on with the New##
The ember router has been on the <a href="" target="_blank">master</a> branch for a while now and it is an emerging pattern that is prone to change and indeed has changed quite a bit in its short existence.  The ember routing solution is one of the main reasons I am drawn to ember and it differs greatly from anything else out there in the **javascript mv*** space.

The Ember router extends the very elegant ember statemanager:
{%codeblock%}
Ember.Router = Ember.StateManager.extend({
});
{%endcodeblock%}
This is the third time I have blogged about ember/sproutcore statecharts/statemanagers/routers etc. (<a href="http://www.thesoftwaresimpleton.com/blog/2012/02/28/statemachine/" target="_blank">here</a> and <a href="http://www.thesoftwaresimpleton.com/blog/2012/04/22/ember-js-routemanager/" target="_blank">here</a>) and I really got excited the first time I came across this new pattern.  The statemachine pattern is a beautiful fit for routing and it is surprising nobody else has come up with it as a solution until now.  

The basic premise is that you describe your application as a hierarchical tree of objects - one object per conceptual state (**Ember.Route** extends **Ember.State**) like below: 
{%gist 3412899 %}

- On **line 1** we provide a subclass of the **Ember.Router** as the **Router** property of the Ember application object which in the example used in this post is named **WZ**.  Naming the router subclass **Router** is a convention that you need to adhere to and I'll explain why later when I explain how an ember application gets initialised under the new regime.
- **Line 2** tells the Ember.Router to log state transitions to the console.
- **Line 3** sets the location property of the **Router** to **hash** which means we will be using url fragment identifiers like **#/post/1** for routing.  You can also specify a value of history which will use the browser's <a href="http://badassjs.com/post/840846392/location-hash-is-dead-long-live-html5-pushstate" target="_blank">pushstate</a> api if one exists.
- On **line 4** a **root** state is provided. As the name **root** suggests, all other routes (or states as I still think of them) will either be direct children of the **root** route or grandchildren of the **root** route.  In true composite fashion, there can only one **root** route.  The **root** route acts as the container for the set of routable states **but** is not routable itself.
- On **line 5** and **line 8** are two such child routes or states of the **root** route named **index** and **home** respectively.  
- Every child route of the **root** route can have a **route** property describing the URL pattern you would like to detect.  On **line 6**, the **index** route has a route property with a value of <strong>'/'</strong> which will correspond to the home page of the application and on **line 9**, the home route has a route property of **/home**.
- On **line 10** is the **connectOutlets** method that you can override in each child route and provide a mechanism for rendering content onto the page as the url changes.  More on this later.

When an Ember application loads, Ember will parse the URL and attempt to find an Ember.Route within the Router's child route hierarchy that matches the url.   Loading the page at '/' which is what the url is when the application is first initialised, will first of all transition the **router** to the first route named **root** and then then to the subsate or child route where it can find a match on the url.  In this case it will find a match on the route property of the **index** route and will transition to that route.  The **index** route simply redirects to the **home** route via the **redirectsTo** directive on **line 7** of the above gist.

Loading the page at the url **#/home** would detect a router path of **root.home** and transition the router to the route or state named **root** and then transition to the substate or child route named **home** which has a route property of **'/home'**.

Below is a test that verifies a root url of '/' will transition to the **home** route:
{%gist 3428957 %}
- On **line 7** we are using the **route** method of the router and passing in a url fragment for the router to try and match on.
-  On **line 8** we are verifying that we have transitioned to the **home** route which is a direct descendant of the **root** route or state.  We verify this by checking the **currentState.path** of the router which uses the dot syntax to signify where we are in the router child route hierarchy.

##Nested Routes##
As you would expect, nested routes correspond to fragments of the url.  Below is a direct descendant of the **root** route named **vault**:
{%gist 3429110 %}
In the above example, the **vault** route has a route of **/vault**, as well as a child state of **new** which in turn has a child state named **step1**.  During routing, these three routes will be composed and a route of **/vault/new/step1** will match the **step1** state.

Below is a test that verifies this:
{%gist 3429205 %}

##State transitions and View Changes##
As each url change transitions the router or statemachine from state to state or route to route, so you would expect what is rendered onto the screen to change also.  Each nested route can take responsibility for what is rendered onto the screen:

Below is the vault route and its child states
{%gist 3429309 %}
And below is a screen grab that outlines which parts of the page are rendered by which route whenever a page is rendered at the following url **#/vault/new/step1**
{%img /images/ember/step.png%}
If the url changes to **#/vault/new/step2** then only the last third of the page will be change when the **step2** route is transitioned to.

##Outlets##
So how is this beautiful tapestry of nested views stitched together?  How do router transitions marry themselves to view changes?  Well, the observant amongst you will have noticed a method named **connectOutlets** that appears in all of the leaf routes (non-root route) of the router.

In order to illustrate how this works, I am going to first refresh our memory of what the router itself looks like and it first child routes that match the home route of '/', which is the url when the application first loads:
{%gist 3412899 %}
As explained earlier, the router will parse the url and try and find a match on the route property of one of the router's child routes or states.  In this case a route will be found on the **index** route which redirects to the home **route**.

On **line 10** of the above, we come to the now infamous **connectOutlets** method (lines 10 - 13 of the above gist ).  Ember provides some nice conventions that are tied to routing.  The **connectOutlets** method will be called when a state or route has been entered and is where you hook up your controllers and views to reflect the change in application state as the url changes. 

The new Ember routing has introduced a number of interesting conventions.  The first one of interest here is that the default handlebars template for the application will be called.....application.handlebars.  The following gist is how the application.handlebars looks for this sample app:
{%gist 3438544 %}
Pretty sparse huh?  All that the above template contains are two **outlets**, one outlet named **nav** and one default outlet.  **Outlets** can be thought of as place holders that you can inject content into.  The connectOutlets method on each child route is the obvious place to connect up these **outlets** with corresponding Ember and controller pairs.

If I can remind everyone of the connectOutlets method:


{%img /images/ember/step.png%}
And this is what the page looks like in the browser
{%img /images/ember/home.png%}
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
