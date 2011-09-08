---
layout: post
title: "Site Refresh With HTML5 and SASS - Part 1"
date: 2011-09-07 10:51
comments: true
categories: SASS HTML5
---
I have been working on <a href="http://www.leadcapturer.com/" target="_blank">this</a> as a side project for some time now and I have decided to take a deep dive into what is available in HTML5 and CSS3 and update the front end code. This yet to be birthed product is written in JRuby and is a rails application.  I have recently been through the pain of upgrading to Rails 3.1 in order to take advantage of the dazzlingly new, shinny and controversial Rails 3.1 <a href="http://guides.rubyonrails.org/asset_pipeline.html" target="_blank">asset pipeline</a>.  I have been using sass and compass for some time and I had previously used the compass blueprint library for layout and utilities.  This feels a bit dated now so it is high time to refresh.  I have previously blogged about sass, compass and blueprint <a href="http://thesoftwaresimpleton.blogspot.com/2010/03/rails-day-6-blueprint-compass-sass-and.html" target="_blank">here</a>. 

I am trying to avoid dissolving into deep depressive rants about things and I could go off on one about the Rails 3.1 abstraction levels reaching their natural meltdown limits with this Rails release.  It is not quite .NET webforms but it is certainly heading in that direction. 

## Markup Refresh

I am going to take this site refresh one blog post at a time.  I want to be sure I remember my transitions.  

The **head** is the only sensible place to start in HTML so here is my new header code which is written in your favourite mark up language and mine <a href="http://haml-lang.com/" target="_blank">HAML</a>:

{% gist 1200025 %}

The above is my main rails layout file.  The following are the main points of interest:

- In HAML the HTML declaration is a two pronged approach.  You first of all place the doctype marker __!!!__ in the .html.haml file:
{% codeblock  %}
 !!!
 %head
{% endcodeblock %}
  Secondly, you need to tell HAML the output format on application start up which is specified in the main rails __environment.rb__ file.
{% codeblock %}
  Haml::Template.options[:format] = :html5
{% endcodeblock %}
  With these options set, we can pass the rendering of the correct __!DOCTYPE__ instruction to the HAML runtime.
  This generates the following HTML declaration:
{% codeblock %}
  <!DOCTYPE html>
{% endcodeblock %}

- On lines 3 and 5 of the gist, we are declaring the single application javascript and css declarations that are generated from the Rails 3.1 <a href="http://guides.rubyonrails.org/asset_pipeline.html" target="_blank">asset pipeline</a>.

- On lines 8 - 11 we are going to take advantage of the <a href="http://code.google.com/apis/webfonts/" target="_blank">Google Web Fonts API</a> and CSS3 to embed some nice looking fonts in the application.  There is a separate declaration and import for each web font that the application will make use of.  More on this in later posts.

- On line 13 we encounter our first new HTML5 tag, the __header__ element.  The semantic **header** tag specifies an introduction, or a group of navigation elements for the document.  

- On line 14, the **hgroup** tag is used to group the document's title and associated subtitles.  The **hgroup** tag can only contain a group of _h1_ to _h6_ elements.

Below is the beautifully indented output of the HAML markup:
{% gist 1202722 %}

## SASS Folder Structure

When I first started using SASS there was nowhere near the level of docs and blog posts available to the end user.  As part of this site refresh, I took a look on <a href="http://github.com/" target="_blank">github</a> at some of the open source projects using SASS to find guidance on how some of the SASS officionados, use SASS.

It is also worth mentioning before proceeding that I use the possibly now old school and pythonesque sass syntax and not the more css like scss syntax.  When I started with sass there was no scss syntax but to be honest, I think .scss was introduced to make sass more designer friendly.  As a developer, I feel more at home with whitespace delimiting my blocks rather than the now horribly out of fashion curly braces.

The screenshot below outlines a popular file organisation pattern for structuring your SASS files:

{% img /images/site-refresh-part1/sass-structure.png %}

The first point of entry is the **application.css.sass** file.  Back in what are probably now classified as the old days, a separate process running the **compass.exe** would watch the sass directory for changes to your sass files.  Any changes were compiled into your ~/public folder.  With rails 3.1, we're getting css and javascript served up through application.css and application.js.  These main application files are really just manifest files that bundle everything in the _stylesheets_ or _javascripts_ folder into one file. 

### First Rails 3.1 SASS Gotcha

Out of the box, the application.css.sass file looks something like this:
{% codeblock %}
/*
 * This is a manifest file that'll automatically include all the stylesheets available in this directory
 * and any sub-directories. You're free to add application-wide styles to this file and they'll appear at
 * the top of the compiled file, but it's generally better to create a new file per style scope.
 *= require_self
 *= require_tree . 
*/
{% endcodeblock %}
The _requite_tree_ statement above tells the system to bundle everything in the stylesheets folder into a single file via the magic of sprockets.

When I first started using this approach in Rails 3.1, I quickly found out that sass variables or indeed sass scopes were not shared across files.  This is because application.css.sass is really for combining plain css files into one via the magic of **sprockets**.  With sass files, you need to use the **@import** directive in the **application.css.sass** file to import additional sass files.  If you are using application.css.scss with *require_tree*, the runtime will still compile your sass and combine the resultant css into one file but there will be no shared scopes for things like variables, mixins etc.  It took me a while to figure that one out.  Remember I mentioned the raised abstraction levels in Rails 3.1?

With that in mind, my application.css.sass file looks like this:
{% codeblock %}
 @import "screen"
{% endcodeblock %}

The **application.css.sass** file is now just a pointer to the main application logic file, **screen.sass**.   

At this time of writing, my **screen.sass** file currently looks like this:

{% gist 1201022 %}

On line 1 we are using the **@import** directive to import the <a href="http://compass-style.org/reference/compass/" target="_blank">compass</a> library into our code.  Using sass without compass is a complete waste of an incredible resource, compass has a wealth of useful features that are hard to ignore.  If you are not using compass with sass, you might as well be using the **less** framework.  Compass has a multitude of mixins, functions and other utilities that make css all of a sudden fun to a css heretic like me.

Lines 2 and 3 of *screen.sass* bring the output or generated styles of two of these compass mixins into the rendered **application.css** output via the sass **@include** directive.  The **@include** directive takes the name of a mixin and optionally arguments to pass to it, and includes the styles defined by the mixin into the current rule.  The following <a href="http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#including_a_mixin" target="_blank">link</a> has some examples that illustrate how they can be used.

Both these mixins generate what are known as css reset rules.  Reset stylesheets try to reduce browser inconsistencies in things like default line heights, paddings, margins and font sizes.  The goal of the reset stylesheet is to give you a completely style free starting point.  For example some browsers indent unordered lists, others do not.  The reset rules level the playing field and gives you a neutral cross browser starting point to add your own.

The first mixin, the **global-reset** mixin is based on <a href="http://meyerweb.com/eric/tools/css/reset/index.html" target="_blank">Eric Myer's 2.0</a> global reset rules.

The second mixin, the **reset-html5** mixin, provides a basic set of HTML5 elements so they are rendered correctly in browsers that do not recognise them and reset in browsers that have default styles for them.

Below is the css that is produced from these mixins:
{% gist 1201540 %}

I think I prefer the 2 lines of sass code to the output above.  There are a number of compass reset mixins that can be utilised in your code.

I think I will leave things here, in the next post I will reveal what is inside the other files in the above **screen.sass** file.  

If you are a web developer and you are not using sass/compass, I have to ask the question....why?