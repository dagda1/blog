---
layout: post
title: "Ember.js - Creating a Custom #link-to Handlebars Helper"
date: 2014-01-11 17:12:55 +0000
comments: true
categories:  JavaScript Ember
---
<!--http://jsbin.com/OnuCaCep/36/edit-->
Following on from my last <a href="http://www.thesoftwaresimpleton.com/blog/2014/01/08/custom-if/">post</a> about how to create a custom **if** helper, I now want to show how to create a custom **link-to** helper.

Ember's routing is arguably the best feature of ember.  Recently while using amazon's S3 file storage web interface where the tree like structure of buckets or folders is implemented client side, I was frustrated to find that the url does not change as you navigate from bucket to bucket which means that you cannot link to a specific bucket or if you refresh the page or naviagate away and then come back you are back at the root bucket.  With ember, you have the ability to make every location on your site linkable thanks to ember's excellent <a href="http://emberjs.com/guides/routing/">routing</a> and the <a href="http://emberjs.com/guides/templates/links/" target="_blank">&#123;&#123;link-to&#125;&#125;</a> helper is a nice convenience that takes a resource and returns a url to that resource.

###The Problem
While iterating over a list of similar model types, you can simply use the **link-to** helper to create links to each item in the list but what if the list contains two or more different types as is illustrated in the gist below where the route's model hook returns a combination of **user** and **contact** types.
{% gist 8377078 %}
One approach would be to do something like this:
{% gist 8377172 %}
The **isUser** condition could compare the context's constructor but this approach is limited as every time you want to link to a different type,  need to update the template. I actually started down this unmaintainable path before souring on the idea as is illustrated in this <a href="http://jsbin.com/OnuCaCep/30/edit" target="_blank">jsbin</a>.

As in my previous posts, the answer to the problem was to create a wrapper around the **link-to** helper and perform a bit of massaging with the arguments array before passing them on to the real **link-to** helper.

Another consideration is that I want to be able to call my custom helper in both the block form and the non-block form.  It is possible to call the link-to helper in its non-block form like this:
{% codeblock %}
&#123;&#123;link-to 'Link Label' 'users' model&#125;&#125;
{% endcodeblock %}
The end result is that I want 	to be able to create the same handlebars expression anywhere in the application and have the helper create the correct link for me.  I want to do this:
{% codeblock %}
&#123;&#123;resource-link-to this&#125;&#125;
{% endcodeblock %}
or this:
{% codeblock %}
&#123;&#123;#resource-link-to this&#125;&#125;
  &#123;&#123;fullName&#125;&#125;
&#123;&#123;/resource-link-to&#125;&#125;
{% endcodeblock %}
And the correct link will be rendered without any thought from me
###The Solution
Here is a <a href="http://jsbin.com/OnuCaCep/34/edit" target="_blank">jsbin</a> of my finished **resource-link-to** helper.

First of all I wanted to create an easy way of getting the corresponding route path from a **DS.Model** type.  I want to be able to get the corresponding route from the type.  If for example, I have an **App.Employee** type then I want to be able to get the string route to that resource which is **employee**. Below is a **humanize** method which does exactly that and is mixed into all **DS.Model** types:
{% gist 8377614 %}
Below is my **resource-link-to** helper that I finally ended up with after much coffee and profanity.  The premise is that I am simply creating a new argument list to pass to the **link-to** helper.
{% gist 8377389 %}
- On **line 3**. I am pulling the resource from the context via the **name** parameter that is defined in the argument list on **line 1**.
- On **line 4** I am using the **humanize** extension function to get the name of the route.
- **Line 6** contains an **if** expression where we branch depending on whether the **resource-link-to** helper is called in its block or non-block formats.  
- **options.fn** will be present if we create the helper in the block form and **options.fn** is the function that will be called to create the text between the blocks.  If the helper is called in its non-block format then we need to pass an extra argument to the real **link-to** helper and on **line 11** we get the new argument from a common property that appears on every model in my example but you could use any logic you like here.
- On **lines 7 and 13** and **lines 8 and 14**, we are passing contextual information for each argument that will be passed to the real **link-to** helper. If you consider that we would be calling the link-to helper like this:
{% codeblock %}
 &#123;&#123;link-to 'Label Text' 'contact'  model&#125;&#125;
{% endcodeblock %}
Then we set the option types and contexts for each argument like this:
{% codeblock %}
options.types = ['STRING', 'STRING', 'ID'];
options.contexts = [this, this, this];
{% endcodeblock %}
- If one of the argument types equates to **'ID'** then it is considered to be a a property of the context at the same array index as that of the **options.contexts** array while 'STRING' means the argument is treated as a literal.
- On **line 18** I am simply calling the real **link-to** helper with the newly created argument list. 

And that is that.  Now we have this wrapper, we can use whatever logic we want to get our route names and link text.

If you have anything to say then please leave a comment below.
