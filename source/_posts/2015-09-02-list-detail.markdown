---
layout: post
title: "Emberjs - Detail view without switching routes"
date: 2015-09-02 21:39:43 +0100
comments: true
categories: Ember, JavaScript
---
Rendering long lists of tabular data on the client is something I have spent a lot of time on and this dates back to my backbone days.  You need to at the very least employ infinite scrolling which opens up other cans of worms.  What if my tabular data has a cell with a link and I click the link and then click the back button.  Unless I spend a lot of engineering effort, I will have lost my place in the infinitely scrolled list.

This was a problem I recently faced on the application that I am working on.  We have a large table of infinitely scrolled rows of contact data that had links to the detail of each contact on each row. We started off with the links redirecting to a new route that displayed the individual contact's details.  This meant that when you clicked the back button from the detail view, the table would be re-renderd from scratch.

I came up with quite a nice and not so technical solution that avoided many dark hours of development.  I came up with the idea of sliding out the view of the contact detail from the same route that contains the table.  This means the user never loses their place in the table.  Throw in some css3 animations and I am very happy with the solution.  Below is an animated gif that shows what I have christened the ```x-drawer``` component:

{% img /images/drawer2.gif %}

Here is a working <a href="http://jsbin.com/kagasu/edit?html,js,output" target="_blank">jsbin</a> which shows the code in action.  The ```x-drawer``` component is a container for other components and is declared in its nested form and any component can be placed inside it:

{% gist 508e73646ba0c69bfa03 %}

I actually use the <a href="https://github.com/emberjs/ember.js/pull/10093" target="_blank">component helper</a> in the production app as I now use the drawer a lot but in the above gist I simply have a property ```showDetail``` that when set to true  will activate the drawer.

The ```x-drawer``` component is actually very simple and the sliding animation is achieved with css3 animations and specifically css transitons that enables the drawer to slide out slowly rather than just appear.  I used a combination of ```translateX``` and ```transform``` to slide out the view.

Another requirement that I had thrust apon me was to make sure the drawer was closed if the user clicked anywhere outside of the drawer.  Below is the code that creates an overlay on the remaining screen real estate that when clicked sends the action to close the drawer:
{% codeblock overlay.js %}
  _setup: Ember.on('didInsertElement', function() {
    var addOverlay, el, self;
    this._super.apply(this, arguments);
    el = this.$();
    self = this;
    addOverlay = function() {
      var overlay, rect;
      rect = el.get(0).getBoundingClientRect();
      overlay = $("<div class='drawer-overlay'></div>");
      overlay.css({
        position: "absolute",
        top: rect.top + "px",
        left: (rect.right - 10) + "px",
        height: rect.height + "px"
        }).appendTo('body');

      return overlay.one('click',function(e) {
        self.send("closeDrawer");
        e.stopPropagation();
        return e.preventDefault();
      });
    };

    el.one($.support.transition.end, addOverlay);

    el.get(0).offsetWidth;
    return el.addClass('open');
  }),
{% endcodeblock %}

This was actually quite a simple solution to a difficult problem.  My first thoughts where that I need to work on the performance of rendering the table but with this solution, the table is always present as we don't have to naviagate away.
