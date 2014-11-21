---
layout: post
title: "Ember.js - Rendering Dynamic Content"
date: 2014-11-18 16:28:08 +0000
comments: true
categories: JavaScript Ember
---
## Warning: The examples use Ember 1.7.1

I'm not going to go into great detail in this post as I think the code examples will be out of date post ember 1.7.1.

I recently had the problem of how to make a complex table reusable accross different datasets.  As each dataset would contain objects with different fields, I would not be able to use the usual handlebars syntax of binding:
{% codeblock %}
&#123;&#123;property&#125;&#125;
{% endcodeblock %}

My solution  was to create a json hash that is similar to the one in the gist below that specified which fields I was binding to and whether or not, I was going to render a simple text field, a link or for complex scenarios a component:
{% gist b417f078e32abcff454c %}
- The address structure on **line 9** is the most basic as it just specifies a property to bind to.
- The structure on **line 4** contains a ```route``` property to signify that I want a link generated.
- The structure on **line 13** contains a ```component``` property that unsurprisingly will render a component.  A component can also take an array of bindings on **line 15** that will have the effect of calling the component like below:
{% codeblock %}
&#123;&#123;full-contact name=name address=address&#125;&#125;
{% endcodeblock %}

Now in my template, I just iterate over this columns collection and either call a handlebars helper that renders the text or link **(line 9)** or I call out to a different helper that will render the component **(line 7)**
{% gist 55b31cfad67e89b7e394 %}

If I am rendering a simple text or link, then I call the helper that is outlined below:

{% gist b14be03dc1b77e2da6cb %}

If I am rendering a component then this helper is called:

{% gist 59a18218dcaf09da60ec %}

Here is a working <a href="http://jsbin.com/fitale/14/edit" target="new">jsbin</a>.

That is all I have to say on the matter but I would love to hear an alternative or better approach to the above.