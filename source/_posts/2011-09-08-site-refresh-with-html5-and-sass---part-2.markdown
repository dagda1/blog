---
layout: post
title: "Site Refresh With HTML5 and SASS - Part 2"
date: 2011-09-08 07:58
comments: true
categories: SASS HTML5
--- 
In the last [post]("http://thesoftwaresimpleton.com/") I took a walk through the basic set up of my sass files that I will be using for the update of my site to the new landscape of html5.

Below is the file layout for my sass files:
{% img /images/site-refresh-part1/sass-structure.png %}
I went on to mention that the **application.css.sass** file above is used as a manifest and I just use it to point to my application point of entry **screen.sass**.

application.css.sass looks like this:
{% codeblock %}
 @import "screen"
{% endcodeblock %}

screen.sass currently looks like this:
{% gist 1201022 %}

In the last post, I explained the how the reset mixins in lines 2 and 3 provide a mechanism for removing browser inconsistencies by a mechanism known as css reset rules.  I now want to explain the contents of the other files that are imported via the sass **@import** directive.

## SASS Partials
All the files that are imported on lines 5 to 12 of screen.sass are sass partials.  Any sass file that is imported as part of a parent or main css file and does not need its own individual css file is what as known as a *partial*.  A sass partial file does not make sense on its own.  In this example, screen.sass will generate one css file that is made up of seven partials.  This is very analgous to rails partials or any partial in any mvc framework.  The convention for a sass partial is to begin the filename with *_* as you can see in the opening image, each file in the partials directory begins with an underscore.  This tells sass that it should not generate an individual css file for the partial and only use it for imports.  To confuse matters, you drop both the *_* and the file extension in the @import statement.

The first import statement on line 5 imports a currently empty file named *_utilities.sass*.  This file will be used for housing any reusable mixins or sass functions that we create on our journey to market.  Next up we have the partial *_colors.sass*.

## SASS Variables
_colors.sass currently looks like this:
{% gist 1205627 %}

The point of *_colors.sass* is to have a central location were we can adjust the colour scheme of the website.  The explicit colour values that will be referenced in other sass files for things like background colours etc. are stored in what are known as sass variables.



