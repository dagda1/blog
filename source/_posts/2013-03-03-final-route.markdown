---
layout: post
title: "Ember.js Routing - The Final Cut"
date: 2013-03-03 22:11
comments: true
categories: JavaScript Ember
---
###WARNING: May cause confusion!###
This is my third post about the now infamous <a href="https://github.com/emberjs/ember.js/" target="_blank">ember.js</a> router.  My initial reaction to the new router was mixed as I quite liked the old router.  One thing that is undeniable about the new router is that it is much easier on the wrist and fingers as it requires writing considerably less code.  That said, the new router is heavily convention based and there were initial periods of bewiderment as I looked at an errorless console and a white screen as my routes were not found.  These periods have receded over the passage of time but there are some traps and pitfalls that catch the unaware.  These things generally only shake out during real world use.  First of all let me set the scene with the model of the sample app that I will use to illustrate the concepts.  
##Model##
I have a simple application that I have been using for my posts that simly allows a user to create a bank of gym exercises that somebody would use while exercising in the gym.  The end goal is to create exercise programs from this bank.  

Every exercise can be categorised into obvious grouping of abdominal, arms, back, chest and leg exercises and below is the group model:
{%gist 5109743 %}
Anybody familiar with an object relational mapper (ORM) will instantly recognise the **DS.hasMany** declaration on line 3 which states that a group can have many exercises associated with it. 

If we look at the Exercise model:
{%gist 5109764 %}
We can see that on **line 5** the other side of the association is declared with the **DS.belongsTo** syntax.
##What about routing?##
I am going to refrain from contrasting the new router with the old because this version seems final for now but as before the router of an ember application is really the focal point of the app.  The router orchestrates state changes in the application via url changes from user interaction.  The router and the various associated states are tasked with displaying templates, loading data, and otherwise setting up application state.  Ember handles this by matching urls or url segments to routes and in order to make this happen, the router has a map function where you can configure this.

Below are the map of relevant urls in this sample app:
{%gist 5109764 %}
As stated previously, we are mapping urls to routes and there a number of ember conventions that dramatically cut down the amount of code required to piece this all togther.  On line 2 of the above gist, we are stating in no uncertain terms that the root or default url **/** will be handled by a route named home.  The last sentence is not entirely true as the **home** string argument that is passed to the route method on line 2 actually does much more than just specifying a route, it also exposes us to the first of the many ember conventions that exist.  The home argument actually specifies that this url should be handled by an ember route named **HomeRoute** and **HomeRoute** will hook up a controller/view/template combination named HomeController, HomeView and a handlebars template named **home.hbs** unless we specity otherwise.
