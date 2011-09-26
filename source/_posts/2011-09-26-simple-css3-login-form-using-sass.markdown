---
layout: post
title: "Simple CSS3 Login Form Using SASS"
date: 2011-09-26 08:34
comments: true
categories: 
---
### Previous Posts
- [Part 1](/blog/2011/09/07/site-refresh-with-html5-and-sass---part-1/)
- [Part 2](/blog/2011/09/08/site-refresh-with-html5-and-sass---part-2/)
- [Part 3: Creating Gradients With Compass](/blog/2011/09/18/creating-gradients-with-compass-and-sass/)

I have been using these posts to cement the knowledge I am gleaning while transforming the markup for a product I am building.  I find trying to transmit the knowledge via blogging is the only way to really learn something or else it is easily forgotten.  In the last few posts I have gone from my project file structure to creating the markup and CSS3 (with the extreme help of sass) to producing the header element.  In this post, I want to outline the steps I took to create the following login page:

{% img /images/simple-form/login.png %}

In the past, I hate to admit that I would have used a table to align the inputs with their labels but this time round I want to upgrade my skill set as I update the site.  With that said, this is the haml that I have created for the above login form:
{% gist 1242152 %}
The above haml generates the following markup:
{% gist 1242169 %}
The only points of interest in the above markup are the new HTML5 attributes that are attached to the input elements on lines 8 and 10 of the above gist.  These attributes are:

- The <a href="http://dev.w3.org/html5/spec-author-view/common-input-element-attributes.html#the-required-attribute" target="_blank">required</a> attribute which as you might have correctly deduced is used to specify that the element requires user input.  Some browsers will alter the default appearance of required elements with a red border for example and most should stop you submitting the form if the mandatory field is left blank.  Remember this is HTML5, things can and will change until ratification.
- The <a href="http://dev.w3.org/html5/spec-author-view/common-input-element-attributes.html#the-placeholder-attribute" target="_blank">placeholder</a> attribute which if set will place default text in an input field if the input element is empty or not in focus.

With the markup out of the way, we can now concentrate on some of the new CSS3 bells and whistles that I have used via sass and compass to create some of the nice effects in the form.

First of all, I want to concentrate on the login form itself:
{% img /images/simple-form/login-form.png %}
Below is the sass that is used to create the sunken effect in the text:
{%gist 1242249 %}
On line 1, is the definition of a mixin that I have used to avoid duplication in the two css selectors that assign styles to the &lt;h2&gt; and &lt;label&gt; elements.  The mixin is used on lines 6 and 12.  It is worth noting that I keep all my colour variable definitions in a separate sass partial file named **_colors.sass**.  This allows me to change the colour scheme in one file and not have to jump around the project files looking for the definitions.  Below is an excerpt from **colors.sass** that shows the variable declarations used in the above gist:
{% codeblock %}
$label-color: #445668
$label-box-shadow: #f2f2f2
{% endcodeblock %}
On line 3 of the _main.sass gist above, I am using the new css3 text-shadow that does what it says on the tin and adds a show effect to an element to make it stand out more.

The from element uses a few of the new css3 techniques to add the new and now infamous css3 border radius property that gives the form the nice rounded corners effect.  It is often jokingly stated that the main purpose of CSS3 is rounded corners.  

But first of all, I want to show how I got this rippled effect on the bottom of the form:
{%img /images/simple-form/form-bottom.png%}
I have used the new css3 box-shadow property to achieve this effect. As the name suggests, the box-shadow property is used to apply an inset or drop shadow to a block element.  It is also possible to add multiple box-shadows to an element as in this example.  As this is a css3 experimental property there are the usual vendor specific prefixes for this property....that is unless you use compass and you can use the <a href="http://compass-style.org/reference/compass/css3/box_shadow/" target="_blank">box-shadow mixin</a> like I am below:
{% codeblock %}
#new_user_session
  @include box-shadow($login-box-shadow-color 0 0 2px, $login-box-shadow-color 0 1px 1px, #fff 0 3px 0, $login-box-shadow-color 0 4px 0, #fff 0 6px 0, $login-box-shadow-color 0 7px 0)
{% endcodeblock %}
I am passing multiple definitions into the mixin that are delimited by commas.  The above sass generates the following css:
{% codeblock %}
#new_user_session {
  -moz-box-shadow: rgba(0, 0, 0, 0.2) 0 0 2px, rgba(0, 0, 0, 0.2) 0 1px 1px, white 0 3px 0, rgba(0, 0, 0, 0.2) 0 4px 0, white 0 6px 0, rgba(0, 0, 0, 0.2) 0 7px 0;
  -webkit-box-shadow: rgba(0, 0, 0, 0.2) 0 0 2px, rgba(0, 0, 0, 0.2) 0 1px 1px, white 0 3px 0, rgba(0, 0, 0, 0.2) 0 4px 0, white 0 6px 0, rgba(0, 0, 0, 0.2) 0 7px 0;
  -o-box-shadow: rgba(0, 0, 0, 0.2) 0 0 2px, rgba(0, 0, 0, 0.2) 0 1px 1px, white 0 3px 0, rgba(0, 0, 0, 0.2) 0 4px 0, white 0 6px 0, rgba(0, 0, 0, 0.2) 0 7px 0;
  box-shadow: rgba(0, 0, 0, 0.2) 0 0 2px, rgba(0, 0, 0, 0.2) 0 1px 1px, white 0 3px 0, rgba(0, 0, 0, 0.2) 0 4px 0, white 0 6px 0, rgba(0, 0, 0, 0.2) 0 7px 0;
}
{% endcodeblock %}
The above css creates the nice looking rippled effect.

Now let us get back to those nice rounded corners that are after all the whole point of css3:
{%img /images/simple-form/box-radius.png%}
The css3 border-radius property is used to give a block element rounded corners and as we cannot be bothered to type all these vendor specific prefixed properties, we are going to use the compass <a href="http://compass-style.org/reference/compass/css3/border_radius/">border-radius mixin</a>.
Below is the border-radius mixin applied to the login form element:
{% codeblock %}
#new_user_session
  @include border-radius(10px)
{% endcodeblock %}
Which generates the following css:
{% codeblock %}
#new_user_session{
-moz-border-radius: 10px;
-webkit-border-radius: 10px;
-o-border-radius: 10px;
-ms-border-radius: 10px;
-khtml-border-radius: 10px;
border-radius: 10px;
}
{% endcodeblock %}
As you can see, compass takes care of the vendor specific pain.

Below is the sass in its entirety that is employed to create the css styling effects of the form minus the inputs.
{% gist 1242453 %}
The only point that I have not covered is the gradient effect that is created using the compass border mixin on line 6 of the above gist.  I went into to linear gradients in the previous [post](/blog/2011/09/18/creating-gradients-with-compass-and-sass/).

##Input Styling
I have used a combination of all the techniques mentioned so far to add some definition to the input elements:
{%img /images/simple-form/inputs.png%}
Below is the sass that is used to create the box-shadow, linear-gradient and border radius effects:
{% gist 1242480 %}

### CSS3 Transitions 
With CSS3 you can now achieve some effects that would have in the past required javascript to achieve.  One of these techniques is known as a <a href="http://www.w3schools.com/css3/css3_transitions.asp" target="_blank">css3 transition</a>.  This allows us to specify that we want change or transition from one style to another.  We can also put a time delay on that transition.

This is impossible to show without a demo and as my site is not on any public facing server yet, I am going to have to point to these <a href="http://24ways.org/2009/going-nuts-with-css-transitions" target="_blank">examples</a>.

In my log in page, I am specifying that I want the input control border to change when the cursor is hovered over the input control:
{%img /images/simple-form/input-hover.png %}
We can also put a delay on the transition which equates to a tasteful delay of a specified time before the transition occurs.

Below is the sass I have used to create this nice time delayed transition:
{% gist 1242522 %}
I am using the compass transition-property and transition-duration mixins to produce this effect.  The compass documentation for the transition mixins can be found <a href="http://compass-style.org/examples/compass/css3/transition/">here</a>.

## Button Effect
Lastly I want to describe how the styling was created for the button:
{%img /images/simple-form/button.png %}
I have used the <a href="http://brandonmathis.com/projects/fancy-buttons/">fancy-buttons</a> library to easily create this effect.

With the library imported, I can simply use the <a href="">fancy-button</a> mixin to generate the style like this:
{% codeblock %}
input[type=submit]
  @include fancy-button(#617798, 14px, 1em, 4px)
  width: auto
  text-transform: uppercase
{% endcodeblock %}
I will leave things here for this article, if you made it to the end of the article then pat yourself on the back!