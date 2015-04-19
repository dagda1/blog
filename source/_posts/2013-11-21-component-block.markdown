---
layout: post
title: "Ember.js Components, wrapping content and context"
date: 2013-11-21 11:30
comments: true
categories: JavaScript Ember
---
**UPDATE:** As of ember 1.10.0, this problem is now solved.  This <a href="http://www.thesoftwaresimpleton.com/blog/2015/04/09/block-params/">post</a> explains how.

I have created <a href="https://github.com/dagda1/ember-autosuggest" target-"_blank">this</a> ember.js <a href="http://emberjs.com/guides/components/" target="_blank">component</a> that allows a user to hook up a data source that can be filtered via user input and then added to a destination:

{%img /images/autosuggest.png%}

You have most likely used something similar when you have been adding tags to some sort of entity.

The component is declared using the hyphonated convention like this:

{% codeblock %}
&#123;&#123;auto-suggest source=controller destination=tags&#125;&#125;
{% endcodeblock %}

This all works great but what if I wanted give users of the component the ability to add their own content to the results?

An example of this might be if you wanted to use the component to add mail recipients to an email form and you wanted to add an avatar to each possible suggestion and selection:

{%img /images/custom.png%}

It turns out that you can also declare <a href="http://emberjs.com/guides/components/" target="_blank">components</a> in a block form whereby components can be passed a <a href="http://handlebarsjs.com/" target="_blank">handlebars</a> teamplate that is rendered inside the component's template whenever a &#123;&#123;yield&#125;&#125; expression appears.

I wrongly assumed that if I added the appropriate &#123;&#123;yield&#125;&#125; statements to my component template then it would be possible for the user to specify their own content like this **AND** the component's context would be mainatined.

{% gist 7581481 %}

All that appeared to be left was to add the&#123;&#123;yield&#125;&#125; statements to my existing template which I did on lines **4** and **15** of the following gist:
{%gist 7580667 %}

And now we get to the meat of the post, the above approach will work fine if you are adding static content to each item but in my case I wanted to bind the appropriate avatar to the **src** attribute of the **img** tag and the appropriate text to the **alt** attribute.  On closer inspection, this was not happening.

It is worth remembering that both my &#123;&#123;yield&#125;&#125; statements are declare in an &#123;&#123;each&#125;&#125; helper:
{%gist 7581348 %}

If I log what the context is in the component's block, I can see it is the context of the parent view and not of the component:
{% gist 7581427 %}
{%img /images/yeild.png%}

After a cry for help, I received some sage advice from <a href="https://twitter.com/marciojunior_me" target="_blank">marciojunior_me</a>, that it is possible to override the **_yield** method of the **Ember.Component** to this:
{% codeblock five.js %}
  _yield: function(context, options) {
    var get = Ember.get, 
    view = options.data.view,
    parentView = this._parentView,
    template = get(this, 'template');
 
    if (template) {
      Ember.assert("A Component must have a parent view in order to yield.", parentView);      
      view.appendChild(Ember.View, {
        isVirtual: true,
        tagName: '',
        _contextView: parentView,
        template: template,
        context: get(view, 'context'), // the default is get(parentView, 'context'),
        controller: get(view, 'controller'), // the default is get(parentView, 'context'),
        templateData: { keywords: parentView.cloneKeywords() }
      });
    }
  },
{% endcodeblock %}

Lines **14** and **15** are the important lines of the above gist where we keep the context of to the current view and not the parentView.

With this in place, all is good again:

{%img /images/emp.png%}

It is worth noting that the **_yield** method is prefixed with the underscore which denotes a private method that you override at your own peril but I have good test coverage in order to catch any future breaks.

I do think this raises an important point about what the context should be when the &#123;&#123;yield&#125;&#125; helper is executed.  I personally think it makes more sense like this as it make sense in this context but I think you could argue either way. 

If you have anything to say on the matter please leave a comment below.
