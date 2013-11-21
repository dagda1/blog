---
layout: post
title: "Ember.js Components, wrapping content and context"
date: 2013-11-21 11:30
comments: true
categories: JavaScript Ember
---
I have created <a href="https://github.com/dagda1/ember-autosuggest" target-"_blank">this</a> ember.js <a href="http://emberjs.com/guides/components/" target="_blank">component</a> that allows a user to hook up a data source that can be filtered via user input and then added to a destination:
{%img /images/autosuggest.png%}

You have most likely used something similar when you have been adding tags to some sort of entity.

The component is declared using the hyphonated convention like this:

{% codeblock %}
&#123;&#123;auto-suggest source=controller destination=tags&#125;&#125;
{% endcodeblock %}

This all works great but what if I wanted give users of the component the ability to add their own content to the results?  

An example of this might be if you wanted to use the component to add mail recipents to an email form and you wanted to add an avatar to each possible suggestion and selection:
<br/>{%img /images/custom.png%}

You can also delare <a href="http://emberjs.com/guides/components/" target="_blank">components</a> in a block form where components can be passed a <a href="http://handlebarsjs.com/" target="_blank">handlebars</a> teamplate that is rendered inside the component's template whenever the &#123;&#123;yield&#125;&#125; expression appears.

Before the change, the template for the component looked like this:
{%gist 7580667 %}