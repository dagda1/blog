---
layout: post
title: "emberjs - arrayComputed"
date: 2014-02-4 08:40:38 +0000
comments: true
categories:  JavaScript Ember
---
**UPDATE:** ```ArrayComputed``` willl soon be deprectated, this <a href="http://www.thesoftwaresimpleton.com/blog/2015/04/18/arraycomputed-refactor/">post</a> expains how to refactor existing ```ArrayComputed``` code.

I'm not entirely sure when this dropped but I am currently on version 1.3.1 of ember and there is a new computed property macro named unsurprisingly **arrayComputed** that is especially designed for working with arrays.  

The basic premise is that you can observe an array and intercept any additions or removals from the array via some useful hooks that gives the developer the opportunity to further mould the data to their needs.

The docs introduce the feature as this:

{% blockquote %}
Creates a computed property which operates on dependent arrays and
  is updated with "one at a time" semantics. When items are added or
  removed from the dependent array(s) a reduce computed only operates
  on the change instead of re-evaluating the entire array.
{% endblockquote %}

One of the problems that this new construct addresses is that when a computed property is declared like below, the entire computed property is recomputed everytime an item is added or removed from the dependency which can be hugely inefficient when dealing with large dependant arrays.  The **arrayComputed** macro uses the now infamous ember.js runloop to coalesce the property changes.
{% codeblock tos.js %}
dealTotals: (function(){
	// do something with deals
}).property('deals.[]')
{% endcodeblock %} 

**Warning:** To take advantage of the **one at a time** semantics, you need to drop the **'.[]'** from the dependant key or the property will recomputed everytime the dependency changes.  If we take the previous example, the declaration would be:

{% codeblock tos2.js %}
dealTotals: Ember.arrayComputed('deals' {
	//implementation
});
{% endcodeblock %}

The best way to illustrate this is with an example, I am going to start with a simple example that works great for a throwaway **jsbin** or very small amounts of data but is pretty impractical in a real world application.  Here is such a working <a href="http://jsbin.com/ilosel/68/edit" target="_blank">jsbin</a> and here is an **arrayComputed** definition that will take advantage of the new construct's hooks to further group the data and provide totals for the grouped data:
{% codeblock group.js %}
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
- **Line 1** declares a function entry point from which an **arrayComputed** property will be returned from.  It takes a dependent key and a callback which is used to return an object by which the dependent array will be grouped by.
- **Lines 2 - 37** is an object literal definition that defines the members that the **Ember.arrayComputed** function expects.
- **Line 3** - declares an initial value for the resulting array.
- **Line 4** - is an **initialize** method that will be called by the framework whenever the computed property is created for real.  I am not taking advantage of that in this example but the example at the end of the post does.
- **Lines 6 - 19** provides the **addedItem** hook that is called every time an item is added to the observed array.  This method uses the **groupBy** callback argument specified in **line 1** that when called returns a key to group the array elements by.  This rest of the **addedItem** method simply creates a group if none exists or increments the count if a group already exists.  What is worth noting here is that we are not actually adding the item to the array but we are adding the group to the array or updating the group if the array already contains it.
- **Lines 20 - 26** provides the **removedItem** implementaiton that is really the converse of **addedItem**.
- **Line 39** actually makes the call to return the **arrayComputed** property.

Below is how you would use the above computed property:
{% codeblock item.js %}
App.CompanyItemController = Ember.ObjectController.extend({
  dealTotals: App.computed.groupable('deals',function(deal){
     return deal.get('state'); 
  })
});
{% endcodeblock %}

You can get quite excited about examples like the one I have just illustrated only to become crestfallen when you apply the above construct to some real asynchronous data.  You might be dealing with promises that have not resolved yet or if you are using **ember-data** then you might call the **groupBy** callback on an item that is still materializing or has its **isLoaded** property set to false which means that the **groupBy** callback will return **undefined** on most occasions.

Below is a real world example that illustrates an approach of how to get round such problems:
{% codeblock grouped2.js %}
  groupedDeals: Ember.arrayComputed('contacts', 'deals.@each.status', {
    initialValue: [],
    initialize: function(array, changeMeta, instanceMeta) {
      array.pushObject(Ember.Object.create({key: 'open',name: 'Open Deals',count: 0,value: 0,deals: Ember.A()}));
      array.pushObject(Ember.Object.create({key: 'closed',name: 'Closed Deals',count: 0,value: 0,deals: Ember.A()}));
      array.pushObject(Ember.Object.create({key: 'lost',name: 'Lost Deals',count: 0,value: 0,deals: Ember.A()}));
    },
    addedItem: function(array, deal, changeMeta, instanceMeta) {
      var contact = deal.get('contact');
      
      var observer = (function(_this) {
        return function() {
          var group, status;
          if (!contact.get('isLoaded')) {
            return;
          }
          contact.removeObserver('isLoaded', observer);
          if (contact.get('company') !== _this.get('model')) {
            return;
          }
          
          status = deal.get('status');
          group = ['closed', 'lost'].contains(status) ? array.findBy('key', status) : array.findBy('key', 'open');
          group.incrementProperty('count');
          group.incrementProperty('value', deal.get('value'));
          group.get('deals').pushObject(deal);
        };
      })(this);
      
      if (!contact) {
        return;
      }
      if (!contact.get('isLoaded')) {
        contact.addObserver('isLoaded', observer);
      } else {
        observer();
      }
      
      return array;
    },
    removedItem: function(array, deal, changeMeta, instanceMeta) {
      var group;
      group = array.find(function(group) {
        return group.get('deals').contains(deal);
      });
      if (!group) {
        return;
      }
      group.decrementProperty('count');
      group.decrementProperty('value', deal.get('value'));
      group.get('deals').removeObject(deal);
      return array;
    }
  }),
{% endcodeblock %}
- On **line 3** I am taking advantage of the **initialize** method to insert some initial values in the resultant array that I can update later.
- On **line 11** I am creating an inline function that does the acutal work of updating the groups that were pushed onto the array in the **initialize** constructor.
- On **lines 33 - 36** I am checking the **isLoaded** property of the inserted item and either creating an observer that will postpone the update of the groups until such a time that **isLoaded** is true or I am calling the function directly.

The **arrayComputed** construct is a real winner for both of the examples in this post. It is especially useful in the second example because creating an observer in a normal computed property would result in a useless result due to the asynchronous nature of the data but with the new construct, we have a reference to the array which we can update when the data is resolved.

One of the challenges of ember or indeed any single page application architecture is to come up with better abstractions for situations like the second example.  I think we need to have less emphasis on examples like the first which make for great demos or jsbins but quickly fall apart when working with real asynchronous data.

My next few posts will be about some of the pitfalls you will face and some solutions when working with real data.
