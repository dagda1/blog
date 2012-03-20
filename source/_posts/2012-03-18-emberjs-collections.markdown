---
layout: post
title: "Ember.js - List DataBinding"
date: 2012-03-18 15:40
comments: true
categories: JavaScript Ember
---
I've been hacking away on quite a detailed results page that uses <a href="http://emberjs.com/">Ember.js</a> and <a href="http://www.handlebarsjs.com">Handlebars</a> templating to render html on the client when it receives JSON objects from a websocket connection on the server.  This turned out to be much more difficult than I first envisioned and I thought I would blog about it in order to help anybody else who struggles with some common problems.  If anybody with greater Ember knowledge than my own can point to a better way then please leave a comment below.
##Scenario##
Below is a screen capture of what I finally ended up with.  
{%img /images/ember/results.png%}
Each row in the table displays the company details that are extracted from a website screen scraping search operation.  Each row can be expanded to display any additional child detail.  The child detail can contain any additional email addresses and contact details that have been scraped from a web site search and belong to the parent company detail row.

##Enter the Ember.ArrayProxy##
One of Ember's strengths is its excellent databinding support and out of the box, it comes with the <a target="_blank" href="http://ember-docs.herokuapp.com/symbols/Ember.ArrayProxy.html">Ember.ArrayProxy</a>.  The ArrayProxy is a construct that wraps a native array and adds additional functionality for the view layer.  In this example, I am going to use the **Ember.ArrayProxy** as a means to publish a collection of objects that I can easily bind the collection to using the handlebars **#each** helper (more on this later). 

It is worth noting that you might also see the **ArrayProxy** out in the wild called the **ArrayController**.  I was informed by somebody on twitter that the ArrayController is now just an alias to the ArrayProxy.  A quick look at the source confirmed this:
{%codeblock %}  
Ember.ArrayController = Ember.ArrayProxy.extend();
{%endcodeblock%}
The ArrayController is a throwback to Ember's sproutcore days and will at some stage be depreciated (I think).  I prefer the name ArrayProxy because it is not a controller in the MVC sense of the word and **ArrayProxy** is a better description of the object's behaviour.

Below is how my extended ArrayProxy ended up:
{% gist 2078762 %}

- The constructor of the ArrayProxy superclass is overriden on **line 2**.
- On line 3 I am calling the base class constructor of the superclass ArrayProxy with **@_super()**.  If you do not call **@_super()** the ArrayProxy will not initialise peroperly.  I only found this out after a lot of head scratching as to why my ArrayProxy was not working.
- On line 5, we are setting the initial array that will be wrapped by the ArrayProxy.  This is done by setting the **content** property. The **content** property must be an object that implements either **Ember.Array** or **Ember.MutableArray**.  For the starting position of our view, we want the ArrayProxy bound to an empty array.  **Ember.A()** will return an empty Ember array.
- On line 7, we create that will reference the handlebars template.  The virtual path to the handlebars template is set on line 8 and on line 10 we attach the initial view to the DOM.
- The **addLead** method on line 12 takes a string of JSON as an argument, creates an Ember model object and uses the **setProperties** method to set multiple properties of the Ember model object with a hash which we create from the passed in string.
Below is how the Lead class looked initially:
{%codeblock%}
Lead.Lead = Em.Object.extend
{%endcodeblock%}
- On Line 15, the newly created object is added to the wrapped array with the **pushObject** method.  This reveals the beauty of the ArrayProxy, when pushObject is called, the bindings will automatically update and render the new lead in the view's output.  The **removeObject** method works the same in reverse.  This requires a lot less boiler plate code than would be required with backbone.js.

##Problem 1 - Numbering the results ##
I will get to the code for the handlebars template shortly, but first I want to show the first problem I came across and that was how to number the results of the table.  Below is an image that highlights the numbering of each row:
{%img /images/ember/results_index.png%}
Below is my handlebars template that will render each object added via the pushObject method. This is the template that the Ember view in the previous *gist* pointed to.
{% gist 2079992 %}
On line 13 of the gist, I am using the **#each** helper which facilitates iterating over an enumerable object.  When I first came across the **#each** helper used in conjunction with the ArrayProxy I thought it would re-render the list each time you called pushObject but this is not the case.  A new item is rendered from **#each** for each new object inserted into the array when pushObject is called.  

I am binding the **#each** helper to an accessible object instance named **Lead.leads_controller** which is actually an instance of the ArrayProxy defined in the previous gist.  The ArrayProxy acts as a normal array and has enumerable behaviour.

When I first wrote this template, I used the **#collection** view helper instead of **#each**.  The **#collection** view helper creates a view for every item in the list.  I was informed again via twitter that the **#collection** view helper would soon be depreciated so I refactored the code to use **#each** and created a separate view for every item in the list which you can see on line 14 with the following line:
{%codeblock%}
#view Lead.Views.Leads contentBinding="this"
{%endcodeblock%}
The contentBinding property is set to **this** which in this context is the current item in the list.  This is a derived view that I have created named **Lead.Views.Leads** and is listed below:
{% gist 2096900 %}
The important bit is the Ember property **adjustedIndex** that is listed on line 4.  

If you are unfamiliar with coffeescript, below is how this would look in javascript:
{%codeblock%}
adjustedIndex: function(){
                 this.getPath('_parentView.contentIndex') + 1;
               }.property();
{%endcodeblock%}
This function defines what is known as an Ember computed property.  Computed properties allow you to treat a function like a property. This is useful because we can treat the computed property like any normal object properties in our handlebars template. In the above example we are accessing the **contentIndex** property of the **_parentView** object and incrementing it by 1.  The fact that the _parentView object is prefixed with an underscore is usually a convention to tell the user of the code that this a private member but I don't think we are doing much harm in this instance.

This seems like a lot of work for this example but computed properties are a powerful feature and can have observable behaviour that allows your template to be auto updating which is one of the main selling points.  We can also now reuse this view to get the adjustedIndex at any point in this template or others.

##Problem 2 - Conditionally Display Results with Handlebars
When it comes to displaying child results, I want to be able to display a message to the user if no results have been found.  For example below is a message I want displayed in the event that no email addresses could be found in the context of the parent row.
{%img /images/ember/noresults.png%}
Handlebars comes with the **#if** helper which does what you would expect with one gotcha.  Handlebars does not support conditional statements like 
{% codeblock %}
#if content.emails.length > 0
{%endcodeblock%}
or 
{% codeblock %}
#if x < y
{%endcodeblock%}
I agree with this because any logic like this should be wrapped up into a helper to make sure the template stays clean.

Below is the part of the template that renders the child emails rows:
{% gist 2127065 %}
- On line 1, we are referring to a property named **hasEmails** of the bound item which in this case is the Lead class I displayed earlier that the handlebars **#if** helper uses as a boolean expression. 
- On line 2 and in the event that **hasEmails** returns true, we are binding the **#each** helper to the emails array property of the parent object.
- On line 3, we are reusing the view we created earlier to utilise the **adjustedIndex** property.
- On line 10 we use the handlebars **else** expansion that can be used with any block helper to represent what the output if the given expression evaluates to a falsey value.

Below is the Lead class I mentioned earlier with two new computed properties named **hasEmails** and **hasContacts** that will be used as boolean expressions by the **#if** helper.  I am using the **Ember.computed** function to create the computed properties this time round but the result is the same as before and it looks a bit neater and a bit more in keeping with coffeescript.
{% gist 2127073 %}
##Conclusion##
Ember certainly feels like a much richer framework than backbone.js which of course means it is a larger framework with more to learn.  There are different abstractions to utilise like the ArrayProxy and I think handlebars is the most feature complete of all the templating libraries I have tried so far.  I am going on to experiment with handlebars partials next.