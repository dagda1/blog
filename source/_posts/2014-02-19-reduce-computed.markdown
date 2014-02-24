---
layout: post
title: "Ember.js - reduceComputed and Property Brace Expansion"
date: 2014-02-19 10:25:56 +0000
comments: true
categories:  JavaScript Ember
---
<!--https://github.com/emberjs/ember.js/pull/2711-->
Following on from <a href="http://www.thesoftwaresimpleton.com/blog/2014/02/04/array-computed/" target="_blank">this post</a> I wanted to quickly mention a nice way to perform summary operations on arrays via the **reduceComputed** computed property.

The docs explain it like this:
{% blockquote %}
Reduce computed properties and computed properties that reduce an enumerable. 
Array computed properties are reduce computed properties whose value happens
to be an array.

Reduce computed properties use one-at-a-time semantics to make computations
from arrays pleasant to write without making it easy to introduce O(n²)
runtime for something that only requires O(n).

They can be chained together and retain their O(n) semantics.
{% endblockquote %}

I've been using ember since **0.9.6** and I have always had a deep fear of the **@each** observer due to the original implementation whereby the array was recalculated after every change that led to O(n²) problems.  I remember having to replace every **@each** in a project with a binary search equivalent.  **reduceCompued** solves this by only dealing with elements that have actually changed.

Below is an example of something that I have just used for a project that I am working on:
{% gist 9089822 %}

Here is a <a href="http://jsbin.com/ilosel/46/edit" target="_blank">jsbin</a> that shows a working example.

One other thing of note is that I am using the new property brace expansion syntax on **line 2** to reference two values which is a nice short hand sugar to reference two dependant keys in the same key:
{%codeblock%}
total: Ember.reduceComputed("deals.@each.{status,value}"
{%endcodeblock%}
This is known as property brace expansion and is so called because it will expand out to form a new dependant key for each item in the comma delimmited list between the braces.

There are a number of **reduceComputed** and **arrayComputed** propeties that come with ember.js out of the box so I decided to cut down on the lines of code in the original implementation by using **Ember.computed.filter**, **Ember.computed.mapBy** and **Ember.computed.sum** to achieve the same result with less code.
{% gist 9090188 %}

In the above gist, I am effectively chaining the results of 3 computed properties together:

- On **line 2** I use **Ember.computed.filter** to only select the items that I am interested in.
- On **line 5**, I map the value I am interested in.
- On **line 6**, I then use **Ember.computed.sum** to reduce the values into one total.

Here is a <a href="http://jsbin.com/ilosel/58/edit" target="_blank">jsbin</a> with a working example.

I believe there are composable computed properties in the pipeline that will make this code even terser so I look forward to that.

**UPDATE:** Thanks to <a href="https://twitter.com/hjdivad" target="_blank">hjdivad</a> who I think implemented the arrayComputed features and who pointed out in the comments that I can use the property brace exapansion with **Ember.computed.filter**.



