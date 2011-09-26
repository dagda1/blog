---
layout: post
title: "Simple CSS3 Login Form Using SASS"
date: 2011-09-26 08:34
comments: true
categories: 
---
### Previous Posts
- [Part 1](/blog/2011/09/07/site-refresh-with-html5-and-sass---part-1/)
- [Part 2](/blog/2011/09/08/site-refresh-with-html5-and-sass---part-2/)
- [Part 3: Creating Gradients With Compass](/blog/2011/09/18/creating-gradients-with-compass-and-sass/)

I have been using these posts to cement the knowledge I am gleaning while transforming the markup for a product I am building.  I find trying to transmit the knowledge via blogging is the only way to really learn something or else it is easily forgotten.  In the last few posts I have gone from my project file structure to creating the markup and CSS3 (with the extreme help of sass) to producing the header element.  In this post, I want to outline the steps I took to create the following login page:

{% img /images/simple-form/login.png %}

In the past, I hate to admit that I would have used a table to align the inputs with their labels but this time round I want to upgrade my skill set as I update the site.  With that said, this is the haml that I created for the above login form:
{% gist 1242152 %}
The above haml generates the following markup:
{% gist 1242169 %}
The only points of interest in the above markup are the new HTML5 attributes that are attached to the input elements on lines 8 and 10 of the above gist.  These attributes are:

- The <a href="http://dev.w3.org/html5/spec-author-view/common-input-element-attributes.html#the-required-attribute" target="_blank">required</a> attribute which as you might have correctly deduced is used to specify that the element requires user input.  Some browsers will alter the default appearance of required elements with a red border for example and most should stop you submitting the form if the mandatory field is left blank.  Remember this is HTML5, things can and will change until ratification.
- The <a href="http://dev.w3.org/html5/spec-author-view/common-input-element-attributes.html#the-placeholder-attribute" target="_blank">placeholder</a> attribute which if set will place default text in an input field the input element is empty or not in focus.

With the markup out of the way, we can now concentrate on some new CSS3 tricks that I have used via sass and compass to create some of the nice effects in the form.