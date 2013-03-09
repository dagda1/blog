---
layout: post
title: "Ember.js Routing - The Final Cut"
date: 2013-03-03 22:11
comments: true
categories: JavaScript Ember
---
###WARNING: May cause confusion!###
This is my third post about the now infamous <a href="https://github.com/emberjs/ember.js/" target="_blank">ember.js</a> router.  My initial reaction to the new router was mixed as I quite liked the old router but over time, I have grown to like it.  One thing that is undeniable about the new router is that it is much easier on the wrist and fingers as it requires writing considerably less code.  That said, the new router is heavily convention based and there were initial periods of bewiderment as I looked at an errorless console and a white screen as my routes were not found.  These periods have receded over the passage of time but there are some traps and pitfalls that catch the unaware.  These things generally only shake out during real world use.  First of all let me set the scene with the model of the sample app that I will use to illustrate the concepts.  
##Model##
I have a simple application that I have been using for my posts that simply allows a user to create a bank of gym exercises that somebody would use while exercising in the gym.  The end goal is to create exercise programs from this bank.  There is an exercise model which describes the individual exercise and a group model that is used to categorise the exercise by abs, arms, back, legs etc..  Both models are listed below for completeness:
{%gist 5109764 %}
##Behold The Router##
I am going to refrain from contrasting the new router with the old because this version seems final and as before the router of an ember application is really the focal point of the app.  The router orchestrates state changes in the application via url changes from user interaction.  The router and the various associated states are tasked with displaying templates, loading data, and otherwise setting up application state.  Ember handles this by matching urls or url segments to routes and in order to make this happen, the router has a map function where you can **map** url segments to routes.

Below are the map of relevant urls in this sample app:
{%gist 5120047 %}
A user will enter an ember site at a specific url or they will interact with a view which will raise an event and cause the url to change.  Entering via a url or the application changing the url will invoke the router which will try and translate the change in url by matching the url with one of the route handlers that you specify.  We have specified in our router that the root url will be handled by a route named home:
{%codeblock%}
WZ.Router.map  ->
  @route 'home', path: "/"
{%endcodeblock%}
Ember routes are also charged with hooking up the correct model, the correct controller and the correct view which will point to a handlebars template.  These will all follow the convention of prefixing the particular object with the string argument that is passed to the route method which in this case is **home**.  The ember runtime will look for a **HomeController**, a **HomeView** or if it cannot find a view, it will look for a **home.hbs** file.  

Now we come to the confusing point, you don't actually have to create any of these objects and in this example, I have not specified a **HomeRoute**, **HomeController** or **HomeRoute**.  When you navigate to the **'/'** root route, Ember will look for a **HomeRoute** and if it does not find it, it will automatically generate a **HomeRoute**.  The same can be said for **HomeController** and **HomeView**.  We can verify this by adding the following expressions to our handlebars template:
{%gist 5122863 %}
Now if we refresh the page, we can actually see what controller and view are backing this route. 
{%img /images/ember/debug.png%}
Ember has automagically generated these objects for us.  The goal of this generation is to eliminate the needless creation of these objects if they are not needed.  We can of course override this behaviour for more complex scenarios.  If I actually needed a controller/view to back this route I could specify the HomeController and HomeView routes that adhere to the naming convention like this:
{%gist 5123020 %}
If we now refresh the page we can see that our objects are hooked up and ready for action:
{%img /images/ember/debug2.png%}
The above is actually a great way of debugging that your views are actually hooked up as I was scratching my head on a number of occasions as I came to terms with the various naming conventions.
##Application Objects##
When your application boots ember will look for an **ApplicationRoute**, **ApplicationController** and an **application.handlebars** file with these objects being generated if they are not specified.  My **application.handlebars** file looks like this:
{%gist 5123106 %}
Anybody familiar with the old router will be more than familiar with the term **outlet**.  An outlet is a placeholder in the template were other templates or content can be injected into.  In the above example there is one named outlet **nav** and a default unnamed outlet. I think I am right in saying (and manybe somebody can correct me) that you do need to create the **application.handlebars** file and it must contain at least one **outlet**.  In our **HomeRoute** example, the contents of the home template will be injected into the default unnamed **outlet** which you can see on line 2 of the above gist.  But what about the **nav** outlet which will render our twitter bootstrap nav bar:
{%img /images/ember/nav.png%}
In order to achieve this, I have created an **ApplicationRoute** that will be executed by the ember runtime when the app first boots and stop the auto generation by the runtime of an **ApplicationRoute**:
{%gist 5123247 %}
The **ApplicaitonRoute** is the correct place for this as the nav will appear on every page.   An ember route has various hooks that you can use to set up other objects or to render additional content.  The **renderTemplate** hook can be used to render a template that differs from the one picked up by convention and it can also be used to render content into other outlets as in this example	.  The current controller and model instances get passed as arguments to this method.  

On line 5 of the above gist we are using the render method to specify:

- Which template we want to render, which in this case is located in our directories folder at **'nav/nav'**
- Which parent template the new content will be injected into which in this case is the **application** template.  This is specified by the **into** property of the options hash.
- Which named outlet of the parent template we want to render the content into which in this case is specified by the outlet property of the options hash and we set that property to **nav**.

One gotcha I had was that I had to call an additional **render** on line 3 or I got an error.  This appeared to be because the main outlet had not been rendered or was undefined.  This was only the case for the **ApplicationRoute**.
##Resources##
If we cast our mind back to our map of urls and routes that we specified:
{%gist 5120047 %}
We can see on line 5 that we are declaring a **resource** mapping, anybody from a rails background will be familiar with this concept but for those who are not, a resource can be thought of a thing or one of the nouns that make up your application.  This usually equates to one of the models and this is no exception as an **exercise** is a pivotal **thing** in our application.  On lines 4 to 6 are the nested routes that belong to the **resource**.  These can be thought of the verbs or the actions that you will carry out on the thing or resource.  We are stating that we will have an **index** route which will list all exercises and two additional routes for creating new exercises and editing existing exercises.  I am not going to get into the restful side of what a resource is because I don't believe it is relevant to client side mvc.
##Index Route##
We have declared that we will have an index route for the exercise resource that will list all exercises:
{%img /images/ember/exercise_index.png%}
In order to achieve this I have created the following 2 routes:
{%gist 5123341 %}
Why 2 routes and not just an index route?  I have created an additional **ExercisesRoute** that is picked up by convention from the declared exercises resource of the router's map in order to use one of the ember route's hooks which is namely **setupController**.  I mentioned at the beginning of the post, that each exercise belongs to a group and I want to have all these groups loaded up and ready for use in the ember data persistence store.  I don't know where or which url the user will use to enter the application but I know I need all the groups loaded and in the store.  The ExercisesRoute is ideal for this because it will be executed **before** any of the child **index**, **new** or **edit** routes that are nested in the **resource** declaration.  I could have used the **ApplicationRoute** for this bootstrapping but I wanted to confine it to the exercises part of the site.

The **setupController** hook is normally used to set up the current controller with the current model so in this case it would be setting the model on the ExercisesController but we are going to gate crash this party as we do not need to specify an ExercisesController and set its model.  On line 3, we are using the **controllerFor** convenience to access the **groupsController** and set its model.  We set its model to all the groups that exist and will be returned from an async call to the server.

On lines 5 - 7 of the above gist, an **ExercisesIndexRoute** is declared with a name that sticks strictly to the naming convention law of the land or it will not be picked up by the runtime.  The **index** route of the exercises resource is nested and the **ExercisesIndexRoute** is named and cased to reflect this:
{%codeblock %}
 Exercises(resource)Index(route)Route
{%endcodeblock%}
We will also need to name our controller and views **ExercisesIndexController** and **ExercisesIndexView** if we want the runtime to pick them up and not auto generate them. 
##Model Hook##
On lines 6 - 7 we use another important hook of the router, namely the **model** hook:
{%codeblock%}
model: ->
  WZ.Exercise.find()
{%endcodeblock%}
The **model hook** is a hook you would use to **set** which model is associated with that url.  A url of **/exercises** should be a listing of all the exercises currently in the database.  In order to achieve this, the model hook is used to query the back end store of all the exercises.  The ember runtime will use this hook to **setup** the controller with the model in the **setupController** hook unless you override it and do something else.
##Templates##
As mentioned earlier, I have an **ExercisesRoute** which equates to a resource in the routers map.  Under this route are the nested actions, **index**, **new** and **edit**.  If we wanted to ensure that all these nested routes are enclosed in the same html elements like below, were the same header will appear in each nested route:
{%img /images/ember/exercises_resource.png%}
To enable this, the **exercises.handlebars** file which is picked up by convention looks like this:
{%gist 5123597 %}
With this in place, all the nested routes will have their content rendered into the above outlet with no extra coding on your part.
##Conclusion##
I have grown to like the new router although initially it was annoying to leave the old router which I liked.  The naming conventions are a double edged sword, on the one hand they drastically cut down on needless ember class declarations but I suspect it might put a few of new people off as it can be difficult to diagnose why your screen is suddenly white and there are no errors in the console.  My advice is to stick with it as ember is a work of art, something quite beautiful and most needed in the javascript world.  On a selfish note, I hope to base the next phase of my career on ember.js work and I hope it has the same growth as rails once did.  This of course is mere speculation on my part, we'll just have to see.

If you disagree with any of this or can point out any mistakes on my part then please leave a comment below.  

I hope ember.js does not confuse you!
