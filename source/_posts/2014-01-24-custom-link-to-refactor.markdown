---
layout: post
title: "emberjs - Custom link-to refactor"
date: 2014-01-24 08:40:38 +0000
comments: true
categories:  JavaScript Ember
---
In a previous <a href="http://www.thesoftwaresimpleton.com/blog/2014/01/11/custome-link-to/" target="_blank">post</a>, I blogged about creating a custom **link-to** helper that takes any ember-data model and creates a link to that model.  I had it all working nicely in this <a href="http://jsbin.com/OnuCaCep/34/edit" target="_blank">jsbin</a> which uses the <a href="http://emberjs.com/guides/models/the-fixture-adapter/" target="_blank">fixture adapter</a> to simulate pulling data from an external service.  I was feeling quite smug after writing this post but I soon got shot down in flames when I plugged the helper into a real appliation that is pulling data from a true asynchronous service.  The picture below illustrates that not all of the sidebar items have a link to signify who the email is from:
<div>{%img /images/isloaded.png%}</div>
I was horrified but it soon be came clear that the devil was at play or to put it another way, asynchronicity was at play.  Below is a refresh of my custom **link-to** helper that I will use to discuss the problem:
{% gist 8594239 %}
The **if** statemnt **Line 6** of the above branches the code execution by deciding whether or not the helper is declared in its block or non-block formats.  The helper in this troublesome example is declared in its non-block format like this:
{% codeblock %}
 &#123;&#123;resource-link-to contact&#125;&#125;
{% endcodeblock %}
As the helper is decared in its non-block format, **options.fn** will be undefined and so the code execution will continue onto **lines 7 - 11**.  The problem stems from how I was getting the anchor label text on **line 11**.  On **line 3**, I get a reference to the ember data model and on **line 11** I was trying to access a generic property named **displayName** that I know exists on all each of the ember data models in this project.  This is problematic when pulling the data from a real external service because the ember data model might not have been loaded or materialized when the helper is trying to access this property as you can see in the image of the console below:
<div>{%img /images/console.png%}</div>
###The solution
The solution involves gaining a better understanding of the the **options** hash that is passed into each handlebars helper.  The code below from the custom helper sets the **options.types** array and the **options.contexts** for each string argument that will be passed to the real **link-to** helper:
{% codeblock %}
if (!options.fn)  &#123;
  options.types = ['STRING', 'STRING', 'ID'];
  options.contexts = [this, this, this];
  args.unshift(name);
  args.unshift(resourceRoute);
  args.unshift(resource.get('displayName'));
&#125;else&#123;
{% endcodeblock %}
The args variabe will look like this when we are finished:
{% codeblock %}
[undefined, "contact", "contact", Object]
{% endcodeblock %}
The first argument is *undefined* for reasons explained above, but each of the first 3 arguments above will marry to the **options.types** array which currently looks like this:
{% codeblock %}
options.types = ['STRING', 'STRING', 'ID'];
{% endcodeblock %}
A *type* of **'STRING'** means it is a string literal and will just be displayed as is but if the **type** is **'ID'** then this denotes that it is a property path and it should be accessed from the context at same array index of the **options.contexts** array.

Armed with this knowledge, the answer was to make **displayName** an **'ID'** type that is a property path to the context.  The updated code looks like this:
{% gist 8594773 %}
On **line 2** of the above gist, I change the first element of the array from **STRING** to **ID** and on **line 6** I push the property name onto the args array that will be passed to the real **link-to** helper.

The only thing left to explain is how the **displayName** property gets rendered into the inner html of the anchor tag that **link-to** helpers produces when the ember-data model is materialized and its **isLoaded** property is true.  The **LinkView** class that the real **link-to** helper creates does this in the code below:
{% gist 8594852 %}
On **line 3** of the above gist, a **linkTextPath** variable is assigned to the **helperParameters.options.linkTextPath** property that will point to the **displayName** property we created in the custom **link-to** helper.  This **linkTextPath** value would be undefined if the option type was still **'STRING'**.

Here is a <a href="http://jsbin.com/OnuCaCep/41/edit" target="_blank">jsbin</a> of the end result.

I was annoyed when my helper did not initially work but as is often the case, it has led me to a greater understanding of the internals of a framework which is very important.

One last note is to point out that this is a great example of why the fixture adapter is not a good simulation of asynchronicity.  It is still very useful but you never run into the occassion where the models have an **isLoaded** set to false because they are loaded straight from the FIXTURES array.  Don't go too deep with the fixture adapter on a real project, you will end up in a fairy tale world that is very different from reality.  I am speaking from heavy experience on this point.