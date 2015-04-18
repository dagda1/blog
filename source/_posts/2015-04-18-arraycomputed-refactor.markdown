---
layout: post
title: "Emberjs - Refactoring away from arrayComputed"
date: 2015-04-18 08:49:11 +0100
comments: true
categories: Ember, JavaScript
---
Word on the emberjs street has stated that the ```reducedComputed``` familty of computed property macros are to be deprecated.  I've used these macros in a number of places in my current contract.  I also previous blogged about ```arrayComputed``` in <a href="http://www.thesoftwaresimpleton.com/blog/2014/02/04/array-computed/" target="_blank">this post</a>.

In the example, I used ```arrayComputed``` to group the number of deals that each particular company had and supplied a total for each grouping.  Here is a working <a href="http://jsbin.com/ilosel/71/edit" target="_blank">jsbin</a> that illustrates the old code.  Each deal has a deal state and I group a particular company's deals by the deal state and give it a total:

{%img /images/old-dealstate.png%}

I will now begin the refactoring.  The first thing I notice after changing the references in the jsbin to point to ```ember 1.11.0``` is that the console is filled with deprecated warnings.  The first warning to get rid of is:

{% blockquote %}
Ember.ObjectController is deprecated, please use Ember.Controller and use `model.propertyName`
{% endblockquote %}

This referers to ```line 1``` of the following gist:

{% gist 5a9c4fac8bd51daa7882 %}

And below is the ```ObjectController``` that the ```itemController``` references in the above template:

{% codeblock old.js %}
App.CompanyItemController = Ember.ObjectController.extend({
  dealTotals: App.computed.groupable('deals',function(deal){
     return deal.get('state');
  })
});
{% endcodeblock %}

I believe controllers are on the condemned list and components will be the main unit of currency so the obvious thing to do is replace the ```itemController``` with a component.  My updated template looks like this:

{% gist 2c03ec29f90480bd2f8b %}

It is worth noting that the ```#each foo in bar``` syntax is also going to be depreceated in favour of the new block syntax that I mentioned in my last <a href="http://www.thesoftwaresimpleton.com/blog/2015/04/09/block-params/" target="_blank">post</a> and you can see how it is used on ```line 2``` of the above.  This pretty much slays most of the warning messages and I now have a component hierarchy as opposed to nested each blocks.

I have the top level company component that is rendered for each company.

{% gist 2c03ec29f90480bd2f8b %}

Below is the x-company template:

{% gist b821273a6a0353fc47cc %}

Each company will render an ```x-deals``` component for each group of deals that are grouped by deal state.  The ```x-deals``` template looks like this:

{% gist ac1fa3be5b53e7a5f358 %}

The ```x-deals``` component contains a collection of ```x-deal``` components.  The ```x-deal``` component's template looks like this:

{% gist a1ca92c6119d47e6db1f %}

###Refactor to the infamous @each Helper

I now want to replace the ```arrayComputed``` macro I created to group the deals.  Below is my original macro definition:

{% codeblock oldArrayComputed.js %}
App.computed = {};

App.computed.groupable = function(dependentKey, groupBy){
  var options = {
    initialValue: [] ,
    initialize: function(array, changeMeta, instanceMeta){
    },
    addedItem: function(array, item, changeMeta, instanceMeta){
      var key = groupBy(item);

      var group = array.findBy('key', key);

      if(!group){
        group = Ember.Object.create({key: key, count: 0});
        array.pushObject(group);
      }

      group.incrementProperty('count');

      return array;
    },
    removedItem: function(array, item, changeMeta, instanceMeta){
      var key = groupBy(key);

      var group = array.findBy('key', key);

      if(!group){
         return;
      }

      var count = group.decrementProperty('count');

      if(count === 0){
         array.removeObject(group);
      }

      return array;
    }
  };
  return Ember.arrayComputed(dependentKey, options);
};
{% endcodeblock %}

Below is how it was used in the old ```itemController```.

{% codeblock olditem.js %}
App.CompanyItemController = Ember.ObjectController.extend({
  dealTotals: App.computed.groupable('deals',function(deal){
     return deal.get('state');
  })
});
{% endcodeblock %}

What was convenient about ```arrayComputed``` was that if any of the company's deal's state attributes changed then the whole property would recalculate and group accordingly.  I wanted to keep this behaviour so I first of all refactored to something that has caused me problems in the past and that is to use the ```@each``` indicator.  The ```@each``` indicator will observe an array for additions and removals AND also it will observe a proprety of each element in the array for changes.

I refactored the ```arrayComputed``` macro to this:

{% codeblock neweach.js %}
App.computed.groupable = function(dependentKey, eachKey,  groupBy){
  return Ember.computed(dependentKey, eachKey, function(){
    var ret = Ember.A();

    this.get(dependentKey).forEach(function(item){
      var key = groupBy(item),
          group = ret.findBy('key', key) || Ember.Object.create({key: key, count: 0, deals: Ember.A()});
      if(!ret.contains(group)) {
        ret.pushObject(group);
      }

      group.incrementProperty('count');
      group.deals.pushObject(item);
    });

    return ret;
  });
};
{% endcodeblock %}

The above code simply returns a standard computed property on ```line 2``` and takes a ```dependentKey``` that is the array being observed and an ```eachKey``` that will contain the ```@each``` indicator.

Below is how the ```x-company``` component that replaced the old ```itemController``` uses the refactored macro.  ```company.deals.@each.state``` will  observe every element of the array for changes to the state attribute.

{% codeblock x-c.js %}
App.XCompanyComponent = Ember.Component.extend({
  dealTotals: App.computed.groupable('company.deals.[]', 'company.deals.@each.state', function(deal){
     return deal.get('state');
  })
});
{% endcodeblock %}

I have created this <a href="http://jsbin.com/yonuri/10/edit" target="_blank">jsbin</a> to show its usage.  I've also updated the bin by adding a dropdown for each deal in the group that can be used to update the deal's state and you can see things recalculate.

{%img /images/ref-dealstates.png%}

###Data down and Actions up
I really was not happy about this refactoring, part of the job of ```arrayComputed``` was to try and nullify the performance pains and problems of ```@each```.  When I first started using ember, ```@each``` was a huge performance drain.  It has improved but I really don't know when it is going to be called by the ember runtime and when it is called is outside of my control.  As I mentioned in this <a href="http://www.thesoftwaresimpleton.com/blog/2015/04/07/observables-evil/" target="_blank">this</a> post, I really want to stop using things that are outside of my control.  I have absolutely no idea when or how many times this macro will be called by the ember runtime and as such, I'm not going to use it.  Experience tells me that I will run into surprises and other headaches if I don't remove it now.

I refactored the computed property macro to a plain old javascript util function:

{% codeblock util.js %}
App._ = {};

App._.groupBy = function(coll, groupBy) {
  var ret = Ember.A();

  coll.forEach(function(item){
    var key = groupBy(item),
        group = ret.findBy('key', key) || Ember.Object.create({key: key, count: 0, deals: Ember.A()});
    if(!ret.contains(group)) {
      ret.pushObject(group);
    }

    group.incrementProperty('count');
    group.deals.pushObject(item);
  });

  return ret;
};
{% endcodeblock %}

The top level ```x-company``` component now explicity determines when the array grouping is recalculated:

{% codeblock recalc.js %}
App.XCompanyComponent = Ember.Component.extend({
  actions: {
    regroupDeals: function() {
      this.groupDeals();
    }
  },

  groupBy: function(deal){
    return deal.get('state');
    },

  groupDeals: function() {
    this.set('dealTotals', App._.groupBy(this.get('company.deals'), this.groupBy));
  },

  _setup: Ember.on('didInsertElement', function(){
    this.groupDeals();
  })
});
{% endcodeblock %}

The ```didInsertElement``` handler on ```line 16``` calls a ```groupDeals``` method that calls the util method.  I also supply a ```regroupDeals``` action on ```line 4``` for down level components to call this action when a deal state changes.

For example, the lowest level ```x-deal``` component will call an action when the dropdown changes:

{% gist a1ca92c6119d47e6db1f %}

This component bubbles the action up when the ```select``` element changes:

{% codeblock x-deal.js %}
App.XDealComponent = Ember.Component.extend({
  actions: {
    changeDealState: function(deal) {
      deal.set('state', this.$('select').val());
      this.sendAction();
    }
  },
{% endcodeblock %}

This is bubbled up to the ```x-deals``` component:

{% codeblock x-deals.js %}
App.XDealsComponent = Ember.Component.extend({
  actions: {
    regroupDeals: function() {
      this.sendAction();
    }
  }
});
{% endcodeblock %}

Which bubbles it up to the ```x-company``` component.

I really don't like having to bubble things up this way but I don't think that there is a better way available in ```1.11.1``` but I'm sure there will be at a future day as this is a bit tedious and coupled.

###Epilogue

I much, much, much prefer my final refactoring as opposed to using the infamous ```@each``` indicator as I am in total control of when things happen.  I can also introduce aynchronicity if the dataset grows and I need to call the server for the groups.  It is up to me when I return the recalculated dataset.

I would take it further and use pubsub to have a reflux like data store do the recalculation as I mentioned in <a href="http://www.thesoftwaresimpleton.com/blog/2015/03/13/ember-reflux/" target="_blank">this post</a> but this will at least do for now.

I think ember should remove the ```@each``` helper because people will use it because of its convenience.

I'm on the fence if I consider computed properties an evil but one thing is for sure, I do not think they are needed as often as I see them used.

I want to explicitly state when my collections/objects are available for re-rendering.  I think this is the most sensible approach and things work much nicer with a top down approach.

Here is an updated <a href="http://jsbin.com/yonuri/11/edit" target="_blank">jsbin</a> with the final refactoring.
