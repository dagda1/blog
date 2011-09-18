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

In the previous two posts, I have started charting the progress of transitioning my yet to born product from old school xhtml, css and javascript into the new world of HTML5 and CSS3.  As the product is not even on the market yet, I am taking the bold step of doing the design myself.  I am a css and design heretic so this is quite a difficult step for me.  On the plus side, I will learn lots on the way no matter what the outcome is. In this article, I want to touch on creating gradients with sass and compass.

A gradient in the context of css is a gradual transition between two colours.  This transition can be transformed through a vertical axis or a horizontal axis.  Below is how my application looks after I have applied the styles that I will create in this article.  
{% img /images/post3/gradients.png %}
I have applied gradients to both the header section and the navigation bar of the html document.  CSS3 comes with a gradient property and most of the modern browsers have their own prefixed slants on this property.  As we are using compass and sass, we can forget about the need to create these vendor specific duplicated linear-gradient properties.  Yet another reason for using compass is that it will do that for us.  The latest version of compass has the excellent <a href="http://compass-style.org/reference/compass/css3/images/" target="_blank">images module</a> and I am going to take advantage of the **background-image** mixin to create the following rule which will generate the relevant vendor prefixed rules.  Below is the rule that will be applied to the new html5 &lt;header&gt; element of our document:
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

It is also worth noting that we are taking advantage of the sass functions lighten and darken to avoid having multiple colour hex values to change.  The above **background-image** mixin will generate the following css:
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
We will use the same technique to apply a gradient to the navigation bar:
{% img /images/post3/nav.png %}
We are going to use the new semantic &lt;nav&gt; element of html5 to mark up the navigation and then apply css rules to.  Below is the haml for the new navigation bar:
{% gist 1225066 %}
And here is the rendered output of the above haml
{% codeblock %}
<nav role="navigation">
    <ul role="user">
      <li>
        <a href="/logout">log Out</a>
      </li>
    </ul>
    <ul role="main-navigation">
      <li>
        <a href="/Home/Index" data-method="get">Capture</a>
      </li>
      <li>
        <a href="/Archive/Index" data-method="get">Archives</a>
      </li>
    </ul>
</nav>
{% endcodeblock %}
I am also using the new html5 semantic attribute <a href="http://www.w3.org/wiki/PF/XTech/HTML5/RoleAttribute" target="_blank">role attribute</a>. HTML5 has many of these new semantic markers that help devices such as screen readers pick out the relevant sections.
 
I want to take this opportunity to show the power of both sass variables and the extremely useful sass functions desaturate, darken and lighten to **DRY** up your css.  Below are  the variable declarations we will be using for the navigation bar:
{% gist 1225069 %}
Only on line 1 do we actually specify any hex value for a variable, in the subsequent variable declarations, we pass in relevant percentage values to the lighten and darken functions to keep everything in ratio. We can change the **$nav-bg** variable and the other variables will adjust accordingly.

A full list of similar sass colour functions can found <a href="http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html" target-"_blank">here</a>

Below is the listing for the **_navigation.sass** partial file that contains the rules for the &lt;nav&gt; tag outlined above:
{% gist 1225073 %}

Besides the **background-image** mixin on line 4 that I described previously, we are taking advantage of a couple of other useful sass mixins:

- <a href="http://compass-style.org/reference/compass/utilities/lists/horizontal_list/" target="_blank">Horizontal List</a> (line 11): which transforms a &lt;ul&gt; list horizontally.
- <a href="http://compass-style.org/reference/compass/utilities/links/link_colors/" target="_blank">link-colors</a> (line 20): Which sets the colors for a link in one mixin.

I will leave things here for this post but I will post about the layout I am going to use for the main section of the page.  I have previously used the compass blueprint library which is a css grid layout framework but the world has moved on and I am going to see what else is available.  If anyone can recommend a more modern approach please leave a comment below.

Please leave any comments below.  Feedback is always appreciated.