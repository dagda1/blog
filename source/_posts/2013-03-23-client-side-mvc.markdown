---
layout: post
title: "Ember.js - Client Side MVC is not Server Side MVC, Erase Your Brain"
date: 2013-03-23 15:05
comments: true
categories: JavaScript Ember
---
{%img /images/shocktreatment.png %}

I'm writing this in the week ending 23 March 2013.  A week that future generations will refer to as hate on ember week.  A lot of people are complaining that ember has a barrier to entry and that barrier is the complexity or the amount of new information that the debutant has to take on board.  So I thought I'd pen a couple of posts that might help set the ground work for understanding ember.

####I think it is very important to grasp this concept before moving on to any of the other concepts in ember such as the router, controller, ItemController etc., etc..####

Ember has a multitude of new concepts to take on board and trying to grasp them all at once can be very overwhelming.

###Client Side MVC is completely different than Server Side MVC###
I am a front end only developer by circumstance and partially because I enjoy coding on the client more.  It was not a choice I made.  I just worked on progressively more javascript heavy apps. 

I have worked with a number of server side mvc frameworks, and as it turns out, these server side incarnations are not actually true mvc in the same sense that the classic smalltalk mvc was.  These server side MVC frameworks are better classified as a <a href="http://en.wikipedia.org/wiki/Model2" target="_blank">model2</a> architecture.  Let us look at the server first.

A typical http request/response of a server side mvc framework is this:

- There is a single point of entry into the system.  All requests are going to come into the system by the browser hitting a route.
- The web server process determines which route it belongs to and dispatches that request to the corresponding controller action.
- The controller will then retrieve the appropriate model before handing the model or some wrapped viewmodel to the view to do the rendering.
- Bottom line is, each GET request usually results in a big blob of something being returned to the browser.  That something could be html, json, xml, text or whatever.

On the server, the collaborating MVC objects only exist for the length the http request.  Http is a stateless protocol and this constrains the real mvc pattern that was popularised by smalltalk.  The views on the server can only receive a request, and dump out some data.

On the client side, there are no such constraints as the objects can live as long as the browser session.  What finally triggered off all the right mental associations about client side mvc for me is the following statement:
{%blockquote%}
a model can notify the view of any changes via the observer pattern.
{%endblockquote%}
This is why comparing ember to rails is not entirely accurate, ember is a spin off from sproutcore which had a stated aim of bringing OSX's <a href="https://developer.apple.com/technologies/mac/cocoa.html" target="_blank">cocoa</a> to the browser.

The best way to flesh this out is with a practical code example.  The gist below contains about the least amount of code I could write to illustrate how a view is updated with changes to the model and vice versa.  You can view this code in the following <a href="http://jsfiddle.net/EhyMR/61/" target="_blankd">jsfiddle</a>.

First up we have the code that will create a very simple object that we can use to show how these updates are reflected.
{% codeblock simple.js %}
window.App = Ember.Application.create();
 
App.Person = Ember.Object.create({
    firstName: 'Paul',
    surname: 'Cowan'
});
{% endcodeblock %}
- On lines 3 to 6 of the above gist, we are creating a very simple object that represents a person. 
- This Person object has **firstName** and **surname** properties.
- This Person object is the **M** for **Model** of the MVC acronym.
- We are creating an ember object and not just a plain old js hash because it needs to be an ember object in order to observe property changes.
- On lines 4 and 5 we are passing in a hash of initial values.

Next we have the markup which is written in an ember flavour of the very powerful client side templating language <a href="http://handlebarsjs.com/" target="_blank">handlebars</a>:
{% gist 5231974 %}
- You can think of the template as the **V** for view of the MVC acronym.
- On **line 1** we have a handlebars expression.  A handlebars expression is enclosed in double curly braces.
- The expression on line 1 points to the Person object's first name property that we created in the previous gist.
- On **line 2** we are creating a view with the handlebars view helper.  We can use the view helper to insert subviews into other views and so aid composability. The view helper takes an object or you can think of it as a path to an object as an argument.  
- On **line 2** we are telling the view helper to render a subview of type **Ember.TextField** which is one of the out of the box view objects that comes with ember.  You might not be surprised to learn that the Ember.TextField renders an input field of type text.

Now we come to the meat and potatoes of the piece thus far and that is the rather peculiar looking **valueBinding** expression on line 2 of the above gist.  I don't think I am exaggerating by saying:
{% blockquote %}
Do not progress any further with ember until you have fully grasped the concept of bindings.  It is the first key checkpoint on your road to Damascus.
{% endblockquote %} 
Ember has a multitude of conventions and this is the first one.  Any ember object property that has the case sensitive suffix **Binding** attached to it will be treated as a special property by the framework.  The **valueBinding** expression below conforms to this rule.  We are setting up a two way communication between two objects with the following expression:
{% codeblock %}
view Ember.TextField valueBinding="App.Person.firstName"
{% endcodeblock %}
The left side of the **valueBinding** expression points to a value of the containing object that is the name minus the **Binding** part.  An input has a **value** attribute and **Ember.TextField** is an abstraction of the input element, so it also has a **value** property.  The right side of the expression can be thought of as a **path** to one of our objects.  The Person object was created as **App.Person** and this object has a **firstName** property.

In short we are wiring together the value of the **firstName** property of the **Person** object to the **value** property of the input.  You can confirm this by typing any text into input field of this <a href="http://jsfiddle.net/EhyMR/61/" target="_blankd">jsfiddle</a> below:
<iframe width="100%" height="300" src="http://jsfiddle.net/EhyMR/61/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
It is worth noting that the Ember Handlebars templates are binding aware and they pick up these changes automagically without any additional code.  Once I learned about bindings, I knew ember was something I should take notice of.

So here we have it, the model is updated from any changes in the view via the observer pattern with no additional code.  No hacky pulling the value out of the DOM and updating the model.  Leave all that behind you, it does not belong here.  The opposite is true also, if the model was updated, the input's value attribute would be updated.

###Computed Properties###
If we wanted to display my full name or a combination of the **firstName** and **surname** properties of our Person object then we could update the handlebars to look like this.
{% gist 5232287 %}
I could do that but then there would be no point in writing this section.  Ember has a special mechanism called <a href="http://emberjs.com/guides/object-model/computed-properties/" target="_blank">computed properties</a> that allow you to create functions that behave like normal properties.  Before going any further, let us update the code to create a computed property that is a combination of the **firstName** and **surname** properties of the Person object.  Below is the updated code:
{% codeblock cp.js %}
window.App = Ember.Application.create();
 
App.Person = Ember.Object.extend({
    firstName: null,
    surname: null,
    
    fullName: function(){
        return this.get('firstName') + " " + this.get('surname');
    }.property('firstName', 'surname')
});
 
App.person = App.Person.create({ firstName: 'Paul', surname: 'Cowan' });
{% endcodeblock %}
We are doing things a wee bit differently here than before:

- On **line 3** we are using **Object.extend** rather than **Object.create**.  This is the preferred route.  You can think of the objects you create with **Object.extend** as the classes from which you create instances of these classes from.  
- On **line 11** we are creating an *instance* of the **Person** class and assigning it to an **App.person** variable which we will reference in our handlebars.
- **Lines 7 -9** define the computed property.  This **fullName** computed property simply concatenates the **firstName **and  **surname** properties into a single output.
- The property expression on **line 9** that is tagged onto the end of the function definition struck me as very odd the first time I came across it.  The property definition on **line 9** takes a comma delimited list of string arguments that equate to ember paths.  These paths can be thought of pointers to ember object properties.  
-  When we add this property syntax onto the end of a function and supply a list of string arguments, we are telling the ember runtime to observe changes in the properties that these paths point to.  In this instance we are stating that whenever anything changes in the **firstName** property or the **surname** property then this computed property needs to recalculate and output its result.  

We need to update our handlebars template to use the computed property:
{%gist 5232372 %}
On **line 1** of the above gist, we are pointing to the **fullName** function that we created on the **App.Person** class.  You can see that we can refer to this function as a normal object property like we did with the firstName and surname properties because the function was extended with the property definition.

Changing either of the text in the inputs causes the computed property **fullName** to recalculate.  

You can verify this below or at this <a href="http://jsfiddle.net/Jr4CB/23/" target="_blank">jsfiddle</a>.
<iframe width="100%" height="300" src="http://jsfiddle.net/Jr4CB/23/embedded/result" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
###Conclusion###
Ember is brimming with a multitude of abstractions that might be different than anything you have come across in the past.  I believe it is better to try and take these concepts in one at a time rather than dive in and get blown away by the routing, controllers, itemControllers and the rest.  I believe it is important to grasp these core concepts first before progressing.  You will use them more than anything else.  Bindings and computed properties are powerful and much more powerful than the examples I have used but we need to get the basics right first.
