---
layout: post
title: "Emberjs - Passing block params from a Component"
date: 2015-04-09 17:51:50 +0100
comments: true
categories: Ember, JavaScript
---
I have previously <a href="http://www.thesoftwaresimpleton.com/blog/2013/11/21/component-block/" target="_blank">blogged</a> about a hack I had to use to get the right context when using a component in its block form.

The example I gave in the post was for an autocomplete component that would give the developer the ability to prepend their own customisation to a selection. The example I gave was prepending an avatar like in the image below.

{%img /images/custom.png%}

The original component could be used in its block form like this:

{% gist 7581481 %}

And here is the original component template file:

{%gist 919daadc5e1a7cda9c05 %}

There is no way in the above to set what the context would be for the code block that is ran when the ```yield``` helper is called.  The context was non-negotionable and the context would be that of the template it was declared in e.g. the controller.  The only way round this was the hack that I described in the previous post.

As of ember ```1.10.0```, it is possible to pass internal values to a down stream scope with the new ```block params``` syntax.

I can refactor my component's template to look like this:

{% gist 1a03df358f3c6b6859b8 %}

- Line 1 uses the ```each``` helper with the new block params syntax to specify a context named ```selection```.
- Line 4 uses the ```yield``` helper to pass the block param to the template or code block that is rendered between the component's definition.

Below is how I can use the block param when the component is declared with a block:

{% gist 6261ff8aff07895d0f10 %}

- line 2 binds the ```selection``` var that is passed to this code block via the ```yield``` helper in the previous code example.
- line 3 puts the var to use.

Here is a <a href="http://emberjs.jsbin.com/vivasa/5/edit?html,js,output" target="_blank">jsbin</a> with a working simpler example.
