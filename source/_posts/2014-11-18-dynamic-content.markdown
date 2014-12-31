---
layout: post
title: "Ember.js - Rendering Dynamic Content"
date: 2014-11-18 16:28:08 +0000
comments: true
categories: JavaScript Ember
---
## Warning: The examples use Ember 1.7.1

I'm not going to go into great detail in this post as I think the code examples will be out of date post ember 1.7.1.

I recently had the problem of how to make a complex table reusable accross different datasets.  As each dataset would contain objects with different fields, I would not be able to use the usual handlebars syntax of binding:
{% codeblock %}
&#123;&#123;property&#125;&#125;
{% endcodeblock %}

My solution  was to create a json hash that is similar to the one in the gist below that specified which fields I was binding to and whether or not, I was going to render a simple text field, a link or for complex scenarios a component:
{% codeblock hash.js %}
App.IndexController = Ember.ArrayController.extend({
  columns: Ember.A([
   {
      heading: "Heading",
      binding: "name",
      route: "company"
    },
    {
      heading: "Address",
      binding: "address"
    },
    {
      heading: 'full',
      component: 'full-contact',
      bindings: ['name', 'address']
    }
  ])
});
{% endcodeblock %}
- The address structure on **line 9** is the most basic as it just specifies a property to bind to.
- The structure on **line 4** contains a ```route``` property to signify that I want a link generated.
- The structure on **line 13** contains a ```component``` property that unsurprisingly will render a component.  A component can also take an array of bindings on **line 15** that will have the effect of calling the component like below:
{% codeblock %}
&#123;&#123;full-contact name=name address=address&#125;&#125;
{% endcodeblock %}

Now in my template, I just iterate over this columns collection and either call a handlebars helper that renders the text or link **(line 9)** or I call out to a different helper that will render the component **(line 7)**
{% gist 55b31cfad67e89b7e394 %}

If I am rendering a simple text or link, then I call the helper that is outlined below:
{% codeblock getproperty.js %}
Ember.Handlebars.registerBoundHelper('getProperty', function(context, property, options) {
  var args, defaults, id, prop;
  
  defaults = {
    className: "",
    context: "this",
    avatar: false
  };
  
  property = Ember.merge(defaults, property);
  prop = context.get(property.binding);
  
  if (Ember.isEmpty(prop)) {
    return "";
  }
  
  if (!property.hasOwnProperty("route")) {
    return new Handlebars.SafeString("<span>" + prop + "</span>");
  }
  
  args = Array.prototype.slice.call(arguments, 2);
  
  options.types = ["ID", "STRING", "ID"];
  options.contexts = [context, context, context];
  
  id = property.context ? "" + property.context + ".id" : "id";
  
  args.unshift(id);
  args.unshift(property.route);
  args.unshift(property.binding);
  
  return Ember.Handlebars.helpers["link-to"].apply(context, args);
});
{% endcodeblock %}

If I am rendering a component then this helper is called:
{% codeblock component.js %}
Ember.Handlebars.registerHelper('renderComponent', function(contextPath, propertyPath, options) {
  var context, helper, property;

  context = Ember.Handlebars.get(this, contextPath, options);
  property = Ember.Handlebars.get(this, propertyPath, options);

  helper = Ember.Handlebars.resolveHelper(options.data.view.container, property.component);

  options.contexts = [];
  options.types = [];

  property.bindings.forEach(function(binding) {
    options.hash[binding] = binding;
    options.hashTypes[binding] = "ID";
    options.hashContexts[binding] = context;
  });

  return helper.call(context, options);
});
{% endcodeblock %}

Here is a working <a href="http://jsbin.com/fitale/14/edit" target="new">jsbin</a>.

That is all I have to say on the matter but I would love to hear an alternative or better approach to the above.