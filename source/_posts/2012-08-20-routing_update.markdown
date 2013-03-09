---
layout: post
title: "Ember.js Routing - The Director's Cut"
date: 2012-08-20 17:32
comments: true
categories: JavaScript Ember
---
**WARN:** This article is now out of date.  Check this <a href="2013/03/03/final-route/">post</a> for a more up to date article on ember routing. 
I've had a number of requests to update <a href="http://www.thesoftwaresimpleton.com/blog/2012/04/22/ember-js-routemanager/" target="_blank">this</a> post which discussed a solution to the lack of routing at that time of writing in <a href="http://emberjs.com/">ember</a>.  This solution used an addon called the <a href="https://github.com/ghempton/ember-routemanager" target="_blank">ember-routemanager</a> from <a href="https://twitter.com/ghempton" target="_blank">Gordon Hempton</a>.    

A lot in Ember has changed since that post and Ember now has its own full blown <a href="http://emberjs.com/guides/outlets/#toc_the-router" target="_blank">routing solution</a> that is not a million miles away from the route manager I blogged about.  I believe <a href="https://twitter.com/ghempton" target="_blank">Gordon Hempton</a> who created the original ember-routemanager is now an ember core member which might explain the stark similarities.  

##Enough of the old and on with the New##
The new ember router has been on the <a href="" target="_blank">master</a> branch for a while now and it is an emerging pattern that is prone to change and indeed has changed quite a bit in its short existence.  The ember routing solution is one of the main reasons I am drawn to ember and it differs greatly from anything else out there in the **javascript mv*** space.

The Ember router extends the very elegant ember statemanager:
{%codeblock%}
Ember.Router = Ember.StateManager.extend({
});
{%endcodeblock%}
The basic premise of the ember routing solution is that you describe your application as a hierarchical tree of objects - one object per conceptual state (**Ember.Route** extends **Ember.State**): 
{%gist 3412899 %}

- On **line 1** we create a subclass of the Ember.Router and assign the reference to a property named **Router** of the Ember application object which in this example is named WZ.  Naming the property **Router** is a convention you must adhere to. 
- **Line 2** tells the Ember.Router to log state transitions to the console.
- **Line 3** sets the location property of the **Router** to **hash** which means that url fragment identifiers like **#/post/1** will be parsed for matches in the routing hierarchy.  You can also specify a value of **history** which will use the browser's <a href="http://badassjs.com/post/840846392/location-hash-is-dead-long-live-html5-pushstate" target="_blank">pushstate</a> api if one exists.
- On **line 4**, a **root** route is created. As the name **root** suggests, all other routes (or states as I still think of them) will be either direct children of the **root** route or grandchildren of the **root** route. The **root** route acts as the container for the set of routable states but is not routable itself.
- On **line 5** and **line 8** are two such child routes or states of the **root** route named **index** and **home** respectively.  
- Leaf routes in the routing hierarchy can have a **route** property describing the URL pattern you would like to detect.  
- On **line 6**, the **index** route has a route property with a value of <b>'/'</b> which is what the url will be when the application first loads.
- On **line 9**, the home route has a route property of **/home**.
- On **line 10** is the **connectOutlets** method that you can override in each child route and provide a mechanism for rendering content onto the page as the url changes.  More on this later.

When an Ember application loads, Ember will parse the URL and attempt to find an Ember.Route within the Router's child route hierarchy that matches the url.   Loading the page at '/' which is what the url is when the application is first initialised, will first of all transition the **router** to the first route named **root** and then to the subsate or child route where the router can find a match on the url.  In this case, the router will find a match on the route property of the **index** route and will transition to that route.  The **index** route in the above example simply redirects to the **home** route via the **redirectsTo** directive on **line 7** of the above gist.

Loading the page at the url **#/home** will transition the router to a substate or route at a path or place in the hierarchy of **root.home**.  This path syntax is useful for testing and what it translates to is that we are currently at the home route which is a child of the root route.  

Below is a test that verifies a root url of '/' will transition to the **home** route:
{%gist 3428957 %}
- On **line 7**, the **route** method of the router is called and a url fragment is passed as an argument for the router to try and match on.
-  **Line 8** asserts that we have transitioned to the expected **home** route which is a direct descendant of the **root** route or state.  We verify this by checking the **currentState.path** of the router which uses the dot syntax to signify where we are in the router hierarchy.

##Nested Routes##
As you would expect, nested routes correspond to fragments of the url.  Below is a direct descendant of the **root** route named **vault** which has a path of **root.vault**:
{%gist 3429110 %}
In the above example, the **vault** route has a route of **/vault**, as well as a child state of **new** which in turn has a child state named **step1**.  If a url of **/vault/new/step1** is requested, all three of these routes will be composed together and all 3 states or routes will be executed in sequence.  Each **connectOutlets** method on each state or route will be executed, giving you a chance to change what is displayed on the page as the url changes and the application state changes.

Below is a test that verifies a url of **/vault/new/step1** transitions to the correct state:
{%gist 3429205 %}

##State transitions and View Changes##
As each url change transitions the router or statemachine from state to state or route to route, so you would expect what is rendered onto the screen to change also.  Each nested route can take responsibility for what is rendered onto the screen:

Below is the vault route and its child states
{%gist 3429309 %}
And below is a screen grab that outlines which parts of the page are rendered by which route whenever a page is rendered at the following url **#/vault/new/step1**
{%img /images/ember/step.png%}
If the url changes to **#/vault/new/step2** then only the bottom segment of the page will be changed when the **step2** route is transitioned to.

##Outlets##
So how is this beautiful tapestry of nested views stitched together?  How do router transitions marry themselves to view changes?  Well, the observant amongst you will have noticed a method named **connectOutlets** that appears in all of the leaf routes (non-root route) of the router.

In order to illustrate how this works, I am going to first refresh our memory of what the router itself looks like:
{%gist 3412899 %}
As explained earlier, the router will parse the url and try and find a match on the route property of one of the router's child routes or states.  When the application first loads or a url of '/' is requested and a route will be found on the **index** route which redirects to the home **route**.

On **line 10** of the above, we come to the now infamous **connectOutlets** method (lines 10 - 13 of the above gist).

Ember provides some nice conventions that are tied to routing.  The **connectOutlets** method will be called when a state or route has been entered and gives the developer an opportunity to reflect the change in application state by injecting content into the place holder of a client side template which is named an **outlet**.

**connectOutlets** works by assigning Ember controller/view pairs, that follow the convention **xxxxController** and **xxxxView** and assigning them to named placeholders called **outlets** that are found in handlebars templates.

When you create and initialize a new Ember application object (more on this later), you must have a controller named **ApplicationController** and a view named **ApplicationView** or Ember will throw an exception.  These two objects will form our first controller/view pair that conforms to the **xxxxController** and **xxxxView** convention.  What is rendered from the **xxxxView** will be placed in the **outlet** or handlebars placeholder. 

Below is my **ApplicationController** from this sample app:
{%codeblock%}
WZ.ApplicationController = Em.Controller.extend()
{%endcodeblock%}

And below is my **ApplicationView** from this sample app:
{%codeblock%}
WZ.ApplicationView = Em.View.extend
  templateName: 'app/templates/application'
{%endcodeblock%}
With these in place, Ember will render the template that is specified at the **templateName** property in the view above.  Below is the template that can be found at the path of the **templateName** property above:
{%gist 3438544 %}
Pretty sparse, huh?  All that the above template contains are two **outlets**, one outlet named **nav** and one default outlet.  **Outlets** can be thought of as place holders that you can inject content into.  The connectOutlets method on each child route is the place to connect up these **outlets** with corresponding Ember view and controller pairs.

Below is a refresh of the **connectOutlets** method for the **/home** route:
{%gist 3438631 %}
The first outlet we connect is the named **nav** outlet (line 4 of the above gist).  The first thing to note on line 4 is that we are accessing the controller via the router with the following syntax:
{%codeblock%}
router.get('applicationController').connectOutlet 'nav', 'navbar'
{%endcodeblock%}
Ember has made the wise choice to move away from accessing objects with the path syntax that was popularised in earlier ember applications when you might have accessed the application controller like this:
{%codeblock%}
WZ.controllers.applicationController
{%endcodeblock%}
And instead objects ending in **Controller** are detected and are assigned as properties of the router when an Ember application is initialized like below:
{%codeblock%}
window.WZ = Ember.Application.create()
WZ.initialize()
{%endcodeblock%}
This means we can pull the controller instances from the router and call the controller's **connectOutlet** method to inject content into the **outlet** placeholders that are specified in the handlebars template.  **connectOutlet** creates a new instance of the provided view class, wires it up with its associated controller, and assigns the new view instance to a property on the current controller.  

What this translates to, is that if we call **connectOutlet** like this
{%codeblock%}
router.get('applicationController').connectOutlet 'nav', 'navbar'
{%endcodeblock%}
We are telling ember to connect the outlet that is named **nav** and that we want to create a new instance of a view called **NavbarView** and wire it up with a controller named **NavbarController**.

The following line from the above gist calls the controller's connectOutlet method and only passes in one argument:
{%codeblock%}
router.get('applicationController').connectOutlet 'home'
{%endcodeblock%}
This tells ember to connect the outlet with no name with a view called HomeView and wire it up with a controller called HomeController.

The HomeView view that was instanciated from the connectOutlet above method points to a handlebars template that contains  a named outlet that we can hook up with the following line:
{%codeblock%}
router.get('homeController').connectOutlet 'bottombar', 'bottombar'
{%endcodeblock%}
The above line grabs a reference to the homeController object via the router, and then call **connectOutlet** which will connect an outlet named **bottombar** with a controller/view pair named BottombarView and BottombarController.  And so we could go on into finer more maintainable detail

And this is what the page looks like in the browser and which  section belongs to which outlet.
{%img /images/ember/home.png%}
I hope you can see that this is a very elegant solution to weaving a rich and maintainable UI from your handlebars templates.

One last thing to mention is that you can supply a **content** for the controller by supplying a final argument for the **connectOutlet** method like this:
{%gist 3441500 %}
On line 4 of the above gist, we are retrieving an array of objects from the remote store and passing that into the connectOutlet method as the last argument.  The content will be assigned to the controller.

##Conclusion##
I am blown away by the elegance of this routing solution and it is the best out there bar none in the javascript mv* space.

I besiege you to get the latest <a href="https://github.com/emberjs/ember.js/" target="_blank">ember</a> from github and take the plunge, you will not regret it.

All the code that is used in these blog posts can be found at this <a href="https://github.com/dagda1/workoutzenith" target="_blank">github repo</a>.

Please feel free to add any comments below.
