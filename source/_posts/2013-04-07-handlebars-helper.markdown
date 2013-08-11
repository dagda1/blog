---
layout: post
title: "Ember.js - Calling a handlebars helper from another handlebars helper"
date: 2013-04-07 12:50
comments: true
categories: JavaScript Ember
---
If you ever have wondered how to call a <a href="http://handlebarsjs.com/" target="_blannk">handlebars</a> helper from another <a href="http://handlebarsjs.com/" target="_blannk">handlebars</a> helper but were unsure how to, then fret no more.  

In case you do not know, <a href="http://handlebarsjs.com/" target="_blannk">handlebars</a> is the templating language of choice for the <a href="http://emberjs.com/" target="_blank">ember.js</a> client side mv* framework.  One of the really great things about handlebars is that it is logic free and you cannot litter your templates with gargantuan expressions.  We have all been guilty of this with rails erb, haml, jsp, asp, asp.net or whatever and let ye without guilt cast the first stone.  Handlebars is the perfect antidote to this simple but very complex erb templating code below:
{% gist 5330234 %}
I came to ember from backbone and I found this logicless approach quite limiting at first but I have grown to love it.  It really does make your templates a hell of a lot cleaner and even if you do get tempted to stray from the chosen path, there is not much you can do apart from create a <a href="http://blog.teamtreehouse.com/handlebars-js-part-2-partials-and-helpers" target="_blank">handlebars helper</a>.

If you are familiar with ember then you will have used handlebars helpers extensively and perhaps even without knowing it, for example you have probably used the view helper to insert new instances of a particular view into a template like this:
{% gist 5330298 %}
You might also have used the **#if**, **#unless** or **#each** block helpers.  You can also create your own helpers that are evaluated in the context of a handlebars template.  In the example I am going to show below, I want to call a helper and pass in a model that contains an image property, the helper will contain all the logic for generating the correct **img** element for a particular model instance.  This logic will also generate a default **img** if the image property of this model instance has not been defined.  Below is a simplified version of how I would call the helper from a template:
{% gist 5330297 %}
This **exerciseImage** helper is being called in an **each** block that is iterating over the model of an **ArrayController**.  The **ArrayController** is made up of **exercise** model instances and the **this** on line 3 of the above gist is the current **exercise** model instance of the iteration loop.

The logic I want to capture is this

- If the model has an image property then I want to display it.
- If the model has no image property then I want to display the relevant image for that exercise group (e.g. abs, arms, legs, back). 
- A group is a **belongsTo** property of the exercise.

I already have a helper that takes a **group** model and displays the correct image and I want to display the **group** image if no image exists for that exercise model.  In other words, I want to call the existing helper from the new helper.  The **group** image handlebars helper looks like this:
{% gist 5330329 %}

- On **line 1** we are registering the helper as a bound helper with the **registerBoundHelper** method.  This means that changes to the bound model will invoke the helper again and the relevant part of the template will get re-rendered again.  I think you will agree that this is pretty damn powerful.
- On **line 3**, we simply pull the name field of the model and use it to generate the correct image **src** on line 7.

Below is the handlebars helper for an **exercise** model instance that will make use of the helper listed above.  If the **exercise** model instance that is the context when this helper is called has no **image** property, i.e. the user has not supplied one, I want to default to the exercise's **group** image.  Below is the code that enables this:
{% gist 5330348 %}

- On **line 4**, I test whether a valid image property was retrieved from the model.
- On **line 5**, I create a new set of arguments from the original set that ignores the first argument.  The first argument would be the exercise model instance and we do not want to pass that to the **groupIcon** helper.
- On **line 6**, I push to the front of the new arguments array, the string path to the **group** model from the current context which is the **exercise** model instance.
- The reason I am pushing the string path as the first argument is because we are simulating how this helper would be called when directly calling it from a handlebars template like this:
{% gist 5331481 %}
- The runtime is expecting a path to some property of the context which in this case is **group** property of the **exercise** model instance.
- On **line 7**, the javascript function object's **apply** method is used:

{% codeblock %}
return Ember.Handlebars.helpers.groupIcon.apply(this, args);
{% endcodeblock %}
**apply** is used to ensure that the method call is made with the correct context and with the newly constructed set of arguments that we created with the string path to the **group** as the first element.

You can also use this technique to apply a layer of indirection to an existing handlebars helper which can be useful to intercept the call and apply some decoration or whatever.

If you have anything to say about this then please leave a comment below.


 
