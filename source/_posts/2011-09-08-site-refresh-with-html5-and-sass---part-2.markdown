---
layout: post
title: "Site Refresh With HTML5 and SASS - Part 2"
date: 2011-09-08 07:58
comments: true
categories: SASS HTML5
--- 

- [Part 1]("/blog/2011/09/07/site-refresh-with-html5-and-sass---part-1/") 

In the last [post]("/blog/2011/09/07/site-refresh-with-html5-and-sass---part-1/") I took a walk through the basic set up of my sass files that I will be using for the update of my site to the new landscape of html5.

Below is the file layout for my sass files:
{% img /images/site-refresh-part1/sass-structure.png %}
I went on to mention that the **application.css.sass** file above is used as a manifest and I just use it to point to my application point of entry **screen.sass**.

application.css.sass looks like this:
{% codeblock %}
 @import "screen"
{% endcodeblock %}

screen.sass currently looks like this:
{% gist 1201022 %}

In the last post, I explained how the reset mixins in lines 2 and 3 provide a facility for removing browser inconsistencies by a mechanism known as css reset rules.  I now want to explain the contents of the other files that are imported via the sass **@import** directive.

## SASS Partials
All the files that are imported on lines 5 to 12 of screen.sass are sass partials.  Any sass file that is imported as part of a parent or main css file and does not need its own individual css file is what is known as a *partial*.  A sass partial file does not make sense on its own.  In this example, screen.sass will generate one css file that is made up of seven partials.  This is very analgous to rails partials or any partial in any mvc framework.  The convention for a sass partial is to begin the filename with a **_** character. If you refer to the opening image, each file in the partials directory begins with an underscore.  This tells sass not to generate an individual css file for the partial and only use it for imports.  To confuse matters, you drop both the *_* and the file extension in the @import statement.

The first import statement on line 5 imports a currently empty file named *_utilities.sass*.  This file will be used for housing any reusable mixins or sass functions that we create on our journey to market.  Next up we have the partial *_colors.sass*.

## SASS Variables
_colors.sass currently looks like this:
{% gist 1205627 %}

The point of *_colors.sass* is to have a central location were we can adjust the colour scheme of the website.  The explicit colour values that will be referenced in other sass files for things like background colours etc. are stored in what are known as sass variables.

One of the glaring misses in CSS is the ability to store reusable values in variables.  This can lead to you having to use potentially faulty search and replace techniques in your text editor to swap hex code values and manage colour palette changes in your stylesheets.  With sass you can assign values to variables, and manage colours, border sizes etc. in a single location.

SASS variables start with the *$* character and can contain any character that is valid in a CSS class name.  In line 1 of _colors.sass, I am storing the hex value of the colour that I will use for the header section of my page.  As _colors.sass is imported in screen.sass, the variables that are declared in _colors.sass will be available in any partials that are imported in screen.sass.  Below is an example of how the sass variable can be used:

{% codeblock %}
body > header
  background: $header-bg
{% endcodeblock %}
Line 3 of _colors.sass contains the following variable declaration:
{% codeblock %}
$subtitle-color: lighten($header-bg, 58)
{% endcodeblock %}
The lighten function that is used to generate the value of $subtitle-color is one of the many utility functions that are included in SASS.  The lighten function unsurprisingly lightens the colour by a percentage value.  In the above example, we can tie the $subtitle-color variable to the $header-bg variable so they will always be in ratio.  A quick look at the <a href="http://sass-lang.com/" target="_bland">sass docs</a> will broaden your knowledge of what sass functions are available.

## Typography

The sass partial file that we are importing is the **_typography.sass** file, which currently looks like this:
{% gist 1218685 %}

On line 1 of _typography.sass, we are declaring a css class with a name of heading and then declaring the font-family we want to use and the fall back options.  It is unlikely you will have heard of the Orbitron font-family but that is because we are importing it from the <a href="http://code.google.com/apis/webfonts/" target="_blank">Google Web Fonts API</a>.  I mentioned this in the previous <a href="/blog/2011/09/07/site-refresh-with-html5-and-sass---part-1/" target="_blank">post</a>.  We can import fonts from the google web fonts api by declaring a separate <link> element for each font we want to import. The declaration for importing the Orbitron font is below:
{% codeblock %}
%link{:rel => "stylesheet", :type => "text/css", :href => "http://fonts.googleapis.com/css?family=Orbitron:regular,italic,bold,bolditalic"}
{% endcodeblock %}

Below is the html that is rendered from the main application layout file **application.html.haml** as it stands thus far:
{% gist 1202722 %}
On line 11 of the _typography.sass file, we are using the css selector **body > header h1** to select the first **h1** element of the new html5 **header** element. On line 13 of _typography.sass, **@extend** is used to tell sass that we want the **body > header h1** selector to inherit all the styles defined in another selector.  This is worth the sass price of admission alone.  The ability to **DRY** up your css by inheriting other selectors rules reduces repeating yourself tenfold. 