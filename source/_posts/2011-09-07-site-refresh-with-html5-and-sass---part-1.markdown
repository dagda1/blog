---
layout: post
title: "Site Refresh With HTML5 and SASS - Part 1"
date: 2011-09-07 10:51
comments: true
categories: SASS,HTML5
---
I have been working on <a href="http://www.leadcapturer.com/" target="_blank">this</a> as a side project for some time now and I have decided to take a deep dive into what is available in HTML5 and CSS3 and update front end code. This yet to be birthed product is written in JRuby and is a rails application.  I have recently been through the pain of upgrading to Rails 3.1 in order to take advantage of the new groovy and controversial Rails 3.1 <a href="http://guides.rubyonrails.org/asset_pipeline.html" target="_blank">asset pipeline</a>.  I have been using sass and compass for some time and I had previously used the compass blueprint library for layout and utilities.  I have previously blogged about sass, compass and blueprint <a href="http://thesoftwaresimpleton.blogspot.com/2010/03/rails-day-6-blueprint-compass-sass-and.html" target="_blank">here</a>. This feels a bit dated now so time to refresh.

## Markup Refresh

I am going to take this refresh one blog post at a time.  The **head** is the only sensible place to start in HTML so here is my new header code which is written in your favourite mark up language and mine <a href="http://haml-lang.com/" target="_blank">HAML</a>

{% gist 1200025 %}

The above is my main rails layout file.  The following are the main points of interest:

- In HAML the HTML declaration is a two pronged approach.  You first of all place the doctype marker __!!!__ in the .html.haml file:
{% codeblock  %}
 !!!
 %head
{% endcodeblock %}
  Secondly, you need tell HAML the output format on application start up which is specified in __environment.rb__.
{% codeblock %}
  Haml::Template.options[:format] = :html5
{% endcodeblock %}
  With these options set, we can pass the rendering of the correct __!DOCTYPE__ to the HAML runtime.
  This generates the following HTML declaration:
{% codeblock %}
  <!DOCTYPE html>
{% endcodeblock %}

- On lines 3 and 5 of the gist, we are declaring the single application javascript and css declarations that are generated from the Rails 3.1 <a href="http://guides.rubyonrails.org/asset_pipeline.html" target="_blank">asset pipeline</a>.  

- On lines 8 - 11 we are going to take advantage of the <a href="http://code.google.com/apis/webfonts/" target="_blank">Google Web Fonts API</a> and CSS3 to embed some nice looking fonts in the application.  We have a separate declaration for each font we are going to use.  More on this later.

- On line 13 we encounter our first new HTML5 tag, the __header__ element.  The semantic **header** tag specifies an introduction, or a group of navigation elements for the document.  

- On line 14, the **hgroup** tag is used to group the document's title and associated subtitles.  The **hgroup** tag can only contain a group of _h1_ to _h6_ elements.

## SASS Folder Structure

When I first started using SASS there was nowhere near the level of docs and blog posts available to the end user.  As part of this site refresh, I took a look on <a href="http://github.com/" target="_blank">github</a> at some of the open source projects using SASS to find guidance on how some of the SASS officiados, such as <a href="http://github.com/imathis" target="_blank">Brandon Mathis</a> use SASS.

The screenshot below outlines a popular file organisation pattern for structuring your SASS files:

{% img /images/site-refresh-part1/sass-structure.png %}