---
layout: post
title: "Ember.js - Calling a handlebars helper from a handlebars helper"
date: 2013-04-07 12:50
comments: true
categories: 
---
I thought I would write a quick post to outline a solution to a repeating problem and that as the title suggests, is the ability to call a handlebars helper from another.  <a href="http://handlebarsjs.com/" target="_blannk">Handlebars</a> is the templating language of choice for the <a href="http://emberjs.com/" target="_blank">ember.js</a> client side library.  One of the really great things about Handlebars is that it is logic free and you cannot litter your templates with gargantuan expressions.  We have all been guilty of this with rails erb, haml, jsp, asp, asp.net or whatever, let ye without guilt cast the first stone.  Handlebars is the perfect antidote to this simple but very complex erb templating code below:
{% gist 5330234 %}
I came to ember from backbone and I found this logicless approach quite limiting at first but I have grown to love it.  It really does make your templates a hell of a lot cleaner and even if you do get tempted to stray from the chosen path, there is not much you can do apart from create a <a href="http://blog.teamtreehouse.com/handlebars-js-part-2-partials-and-helpers" target="_blank">handlebars helper</a>.

If you are familiar with ember then you will have used handlebars helpers extensively and perhaps without even knowing it, for example you have probably used the view helper to insert new instances of a particular view into a template:
{% gist 5330298 %}
You might also have used the **#if**, **#unless** or **#each** block helpers.  You can also create your own helpers that are evaluated in the context of a handlebars template.  In the example, I am going to show below, I want to pass in a model into the handlebars helper and the helper will contain the logic to select the correct image for that model and output the correct html.  Below is a simplified version of how I would call the helper from my template:
{% gist 5330297 %}
This template is being called in an each block which is iterating over the model of an **ArrayController**.  The ArrayController has a model of a list of exercise model instances and the **this** on line 3 of the above is the current model in the iteration loop.

The logic I want to capture is this

- If the model has an image property then I want to display it.
- If the model has no image property then I want to display the relevant image for that exercise group (e.g. abs, arms, legs, back). 
- A group is a **belongsTo** property of the exercise.

I already have a helper that takes a group model and displays the correct image and I want to display the group image if no image exists for that exercise model.  In other words, I want to call the existing helper from the new helper.  The group image handlebars helper looks like this:
{% gist 5330329 %}

- On **line 1** we are registering this helper as a bound helper with the **registerBoundHelper** method.  This means that changes to the model will mean this part of the template gets re-rendered if the model changes.  I think you will agree that this is pretty damn powerful.
- On **line 3**, we simply pull the name field of the model and use it to generate the correct image **src**.

Below is how I ended up calling this from my the **exerciseImage** helper that I created:
{% gist 5330348 %}

- On **line 4**, I test whether a valid image property was retrieved from the model.
- On **line 5**, I extract a new set of arguments that ignores the first argument which was the **this** or the exercise model was passed as an argument to the helper with the **exerciseImage this** call from the original handlebars template.
- On **line 6**, I push to the front of the new arguments the string path to the group model which is a **belongsTo** property from the current exercise model that we will need in the group helper.  You can think of this path as **exercise.group**.
- I then use the javascript function object's **apply** method to ensure that the correct context is used

You can also use this technique to apply a layer of indirection to existing handlebars which can be useful to intercept the call to one of the other helpers.

If you have anything to say about this then leave a comment below.


 
