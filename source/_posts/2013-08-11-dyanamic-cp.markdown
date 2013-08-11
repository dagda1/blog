---
layout: post
title: "Ember.js - Creating computed properties at runtime"
date: 2013-08-11 19:51
comments: true
categories: JavaScript Ember
---
I came across an interesting problem not so long and that I think is worthy of a post as I could not find anything meaningful about it online when I came across the problem.

I am currently working on a CRM app that tracks the lifecycle of deals from cradle to grave.  A deal in our domain might start life in an *initiated* or *opened* state to signify the sourcing of a client and getting their contact details.  A deal can then transition into other states before it finally reaches a *closed* or *lost* state.  We call this the deal pipeline and the application has a view that displays all these states with a total beside each state which equates to the total of deals that are currently in that state, something like this
<br/>{%img /images/pipeline.png%}
I wanted to create a computed property for each state that would update as deals are created.  I wanted to do this but there was a problem.  The deals states are not static and are in fact created by the user and as such, not known until runtime.  The only way to create a computed property for each state was to do it at runtime.  

I did a quick google but nothing came back so I had a look at the ember tests to see how they were testing computed properties and came across examples like this:
{% gist 6206499 %}
It turns out that you can use the EMAScript 5 <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty" target="_blank">defineProperty</a> to create a computed property, just in the same way as you create any property using *defineProperty*.  For those unfamiliar with *defineProperty*, the method allows you to define a new property on an object (or change the descriptor of an existing property).  It turns out that ember has its own version of *defineProperty* that also accepts computed properties.

Armed with this knowledge, this is how my code ended up:
{% gist 6206559 %}

I have created this <a href="http://jsbin.com/ilosel/16/edit" target="_blank">jsbin</a> that roughly shows the same effect.

If you disagree with any of the above or can suggest a better way then please leave a comment or fork the jsbin and improve it.	