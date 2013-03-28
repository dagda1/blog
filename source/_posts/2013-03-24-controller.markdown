---
layout: post
title: "The Ember Controller - Same same, but different - Part 1 - the ObjectController"
date: 2013-03-24 19:00
comments: true
categories: JavaScript Ember
---
I have travelled around the beautiful country of Vietnam twice and they have a rather quaint saying:
{%blockquote %}
Same same but different
{% endblockquote %}
I think this saying is descended from the ember.js controller.  The ember.js controller is similar in many respects to a controller object that you might find in a server side framework like rails but at the same time it is quite different.  Let us flesh this out.

In my last <a href="http://www.thesoftwaresimpleton.com/blog/2013/03/23/client-side-mvc/">post</a> I made this statement:
{%blockquote%}
a model can notify the view of any changes via the observer pattern.
{%endblockquote%}
As it turns out, the above statement is not entirely true or at least, I purposely omitted some facts and not all the actors of the piece received their dues.  A model does indeed notify the view of any changes but it does so by way of a middle man who sits in between the view and the model.  This middleman is of course the ember.js <a href="http://emberjs.com/guides/controllers/">controller</a>.  The ember api docs portray this shadowy middleman as this:
{%blockquote %}
In Ember.js, controllers allow you to decorate your models with display logic. In general, your models will have properties that are saved to the server, while controllers will have properties that your app does not need to save to the server.
{%endblockquote%}
As always the best way to illustrate this is with some code.  Let us look at an example of what would happen if we did not have a controller.

Let us say we had an ember-data Employee object like this:
{% gist 5257190 %}
The above model has 3 simple fields of **firstName**, **surname** and **age** and a computed property of **fullName**.

Now let us say we are working on an application where we can change the status of an employee to retired or we can reinstate retired employees.  We might have a page like this to set an employees status to retired:

{%img /images/employee/retire2.png %}

You will have to forgive the crude html but it is a jsfiddle that you can see in it's entirety <a href="http://jsfiddle.net/dagda1/CFyVH/3/" target="_blank">here</a>.   You might also have a page that allows you to make that employee active again which would look like this:

{%img /images/employee/active2.png %}

Now what if we wanted to capture the fact that we had selected the contact.  Now the whole point of client side MVC is that we want to use the model, we don't want to be mucking about the DOM to check if an item is selected, we want to use our rich abstraction to capture this check, so we might to might add an **isChecked** property to our model like on line 8 of the gist below.  
{% gist 5257792 %} 
We could then bind (refer to my last <a href="http://www.thesoftwaresimpleton.com/blog/2013/03/23/client-side-mvc/">post</a>) this **isChecked** property to the checkbox in the view like this:
{% gist 5257860 %}
On line 3 of the above gist we have an **checkedBindig="isChecked"** that will take care of changing the model's **isChecked** property without any DOM manipulation.

Now we can capture the whether **isChecked** for this item on the model.  All is great, slap yourself on the back, you are a client MVC wizard.  But hold on, do not gather your wife and kids around you and tell them of your gallant exploits just yet.  

Using the gist below, take the following steps:

- Click the active link
- Check the checkbox
- Click home
- Click retired
- Recoil in horror as the checkbox is already checked.  You did not check this checkbox, why is it checked???  
- Scream that you want to use jQuery and jQuery only as this is now getting complicated.

<iframe width="100%" height="300" src="http://jsfiddle.net/dagda1/CFyVH/3/embedded/result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

This is of course because the model instance is shared throughout the whole application and **isChecked** is a two way binding.  When we set isChecked on the model, we are setting it throughout the application.  There is of course an answer to this.

##Enter the ObjectController##

If you cast your mind or eye back to the beginning of this post, I quoted the following from the docs
{%blockquote %}
In Ember.js, controllers allow you to decorate your models with display logic.
{%endblockquote %}
Controllers in ember act as proxies for their underlying model.  We can then decorate the model with display logic.  In this less than perfectly crafted example, the **isChecked** property is exactly the type of display logic we want to decorate our model with.  You can think of the **isChecked** property as an example of **application state** that should only exist for the lifetime of the current view and not be persisted along with the model's long lived properties.

As stated above, controllers in ember are proxies for their underlying template.  We can access the model's properties through the controller because the controller just acts as a pass-through for the model properties.  Ember has a number of different flavours of controller, there is the **ArrayProxy** when your model is a list of models or in this case, there is the **ObjectController** when you are dealing with a single object.  Templates are always connected to controllers and not models.  

We will now use the **ObjectController** to solve this troubling situation.  Currently our router looks like this:
{%gist 5260930 %}
We have one resource which is the **Employee** resource that we are dealing with and two routes that correspond to the two views with the checkboxes, **retired** and **active**.  Ember is simply brimming with conventions and the objects at your disposal for a particular url follow the convention **xxxRoute**, **xxxController**, **xxxView** or they will be generated for you to eliminate a lot of the boilerplate code that used to exist. As we have not defined and **ActiveController** or a **RetiredController**, ember will generate these for us.  

The route handler is the place where you set the underlying model for a controller as the route handler's job is to translate a specific url into a model object.  There are a number of hooks that serve this purpose and both routes in the above gist are using the **model** hook to specify which model is associated with this url.  In both cases, we are hardcoding which model is going to be associated with each url.  We are now going to change things by specifying a controller for each of these routes that will override the automagically generated controllers.  Following the convention of **xxxController**, we create the following two controllers:
{% gist 5260990 %}
Our work here is done, take the same actions as before on the jsfiddle below:
<iframe width="100%" height="300" src="http://jsfiddle.net/dagda1/CFyVH/4/embedded/result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
Our work here is almost don, we can decorate the model with application state that is not persistent outside of the controller's scope.

###Conclusion###
This is a fairly stupid example but I hope this clarifies why we need a controller and what its job is.  You can now call your wife and children to gather round, daddy has an announcement, daddy is a hero!

I'll cover the **ArrayProxy** next.


http://jsfiddle.net/dagda1/CFyVH/3/


