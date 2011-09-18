---
layout: post
title: "Creating Gradients With Compass and SASS"
date: 2011-09-18 12:08
comments: true
categories: html5 sass compass
---
### Previous Posts
- [Part 1]("http://www.thesoftwaresimpleton.com/blog/2011/09/07/site-refresh-with-html5-and-sass---part-1/")
- [Part 2]("http://www.thesoftwaresimpleton.com/blog/2011/09/07/site-refresh-with-html5-and-sass---part-2/")

In the previous two posts, I have started charting the progress of transitioning my yet to born product from old school xhtml, css and javascript into the new world of HTML5 and CSS3.  As the product is not even on the market yet, I am taking the bold step of doing the design myself.  I am a css and design heretic so this is quite a difficult step for me.  I will learn lots on the way if nothing else. In this article, I want to touch on creating gradients with sass and compass.

A gradient in the context of css is a gradual transition between two colours.  This transition can be transformed through a vertical axis or a horizontal axis.  Below is how my application looks after I have applied the styles that I will create in this article.  
{% img /images/post3/gradients.png %}
I have applied gradients to both the header section and the navigation bar of the html document.  CSS3 comes with a gradient property and most of the modern browsers have their own prefixed slants on this property.  As we are using compass and sass, we can forget about the need to create these vendor specific duplicated linear-gradient properties.  Compass will do that for us.  The latest version of compass has the excellent <a href="http://compass-style.org/reference/compass/css3/images/" target="_blank">images module</a> and I am going to take advantage of the **background** mixin to create the following rule which will generate the relevant vendor prefixed rules.  Below is the rule that will be applied to the new html5 &lt;header&gt; element
{% codeblock %}
body > header
  background-color: $header-bg
  @include background-image(image-url('noise.png'), linear-gradient(darken($header-bg, 20), $header-bg, lighten($header-bg, 11)))
{% endcodeblock %}
I am using the **background-image** mixin to generate the linear-gradient as prescribed in the latest <a href="http://compass-style.org/CHANGELOG/" target="_blank">compass docs changelog</a> which states:
{% blockquote %}
The linear-gradient and radial-gradient mixins have been deprecated. Instead use the background-image mixin and pass it a gradient function. The deprecation warning will print out the correct call for you to use.
{% endblockquote %}
We are also passing in a fallback image if the browser does not support linear-gradient.

For IE6 and IE7 there is the <a href="http://compass-style.org/reference/compass/css3/images/#mixin-filter-gradient" target="_blank">filter-gradient</a> mixin.  I will probably create an **IE &lt; 8** conditional stylesheet and try this out at a later stage.

It is also worth noting that we are taking advantage of the sass functions lighten and darken to avoid having multiple colour hex values to change.  This will generate the following css:
{% codeblock %}
/* line 1, /Users/paulcowan/projects/leadcapturer/app/assets/stylesheets/partials/_header.sass */
body > header {
  background-color: #6a85af;
  background-image: url(/assets/noise.png), -webkit-gradient(linear, 50% 0%, 50% 100%, color-stop(0%, #acbbd3), color-stop(50%, #6a85af), color-stop(100%, #4f6992));
  background-image: url(/assets/noise.png), -webkit-linear-gradient(#acbbd3, #6a85af, #4f6992);
  background-image: url(/assets/noise.png), -moz-linear-gradient(#acbbd3, #6a85af, #4f6992);
  background-image: url(/assets/noise.png), -o-linear-gradient(#acbbd3, #6a85af, #4f6992);
  background-image: url(/assets/noise.png), -ms-linear-gradient(#acbbd3, #6a85af, #4f6992);
  background-image: url(/assets/noise.png), linear-gradient(#acbbd3, #6a85af, #4f6992);
}
{% endcodeblock %}
##Navigation Bar##
We pretty  much use the same technique for the navigation bar:
{% img /images/post3/nav.png %}