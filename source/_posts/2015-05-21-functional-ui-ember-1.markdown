---
layout: post
title: "Emberjs - Avoiding Side Effects....or at least trying"
date: 2015-05-21 20:23:00 +0100
comments: true
categories: Ember, JavaScript
---
I've been using ember for 2+ years and the more I deal with a single page application that exists in memory for an indeterminate length of time, the more tenets of functional programming I embrace to bring some sanity to the chaos.  The problem with ember is that it is more class or object based than functional.

The move to ember 2.0 will see ember become more functional but I think it needs to go further and drop a lot of the class based paradigms to make it more robust.

I'll quickly run through the important tenets that could make your code more robust if you can employ them:

###Function Purity
You should always strive to make your functions as pure or side effect free as possible.  Wikipedia gives the following definition for a pure function:
{% blockquote %}
A pure function is a function where the return value is only determined by its input values, without observable side effects
{% endblockquote %}
Another way of expressing the above is that a *pure function* will always return the same value when given the same arguments.

Below is the most basic of basic examples of a pure function
{% codeblock pure.js %}
function addOne(num) {
  return num + 1;
}
{% endcodeblock %}
This function is pure because it will always return the same result, given the same input.  It only reads local variables and it only writes to local variables.  This is also sometimes referred to as *referential tranparency* because the expression can be changed with its value without effecting the behaviour of a program.  For example if we used the ```addOne``` function like this:
{% codeblock ref.js %}
addOne(1) + addOne(1); //=> 4
{% endcodeblock %}

We can replace the expression with its value and still get the same result:
{% codeblock replace.js %}
2 + addOne(1); //=> 4

2 + 2; //=> 4
{% endcodeblock %}

A simple example of an impure function is below:
{% codeblock sideeffect.js %}
var number = 2;

var sideEffect = function(n) {
  number = n * 3;
  return number * n;
}

sideEffect(3);
{% endcodeblock %}
The *side effect* of this function is that it changes the value of the ```number``` variable each time it is executed.  The example above is not *referentially transparent* because if we call ```sideEffect``` with 3 we would get a different result each time.

On the surface dealing with only pure functions is very limiting, we would need to avoid things like ```new Date``` or ```Math.Rand``` because they always return a different value.  ```console.log``` is also on the banned list because it accesses a global variable.  In fact, any function that takes an object as an argument is potentially subject to impurity. The DOM is one big huge stateful side effect waiting to happen.

The bottom line is that if you are dealing with javascript, you will always be dealing with some degree of impurity or our programs would not be very useful.  What we need to do is try and minimize this.

Ember's Handelbars helpers can be pure functions but I have seen an RFC recently that wants to add all sorts of state and life cycle events which is something that made me shudder.  I worry that if this rfc is pushed through then it will complicate something that is very useful and can be side effect free.

####Purity and Idempotence
Idempotence entered the vernacular or at least resurfaced when REST went mainstream.  An idempotent operation is one that has no additional effect if it is called more than once with the same input parameters.  For example moving an item from a set can be considered an idempotent operation or deleting a record by GUID is idempotent becuase the row stays deleted.

Purity and idempotence are completely orthogonal, a pure function does not have to be orthogonal and vice versa.  An idempotent function can cause idempotent side effects, a pure function cannot.

####Immutability
In functional prograaming, the ideal situation is that there is never mutation.  I blogged <a href="http://www.thesoftwaresimpleton.com/blog/2015/03/13/ember-reflux/" target="_blank">previously</a> about some of the benefits of immutability and gave an example of how it might be used.

One of the traps I fell down when I first started developing with ember was to observe changes on mutable arrays like this.

{% codeblock observe.js %}
arrayDidChange: Ember.computed('myArray.[]', function() {
  // do stuff
}
{% endcodeblock %}
I see this type of code in a lot of stackoverflow answers and blog posts but I now consider this to be an antipattern.  Two way databinding was one of the things that first drew me to ember but ended up causing me no end of pain.  I think two way databinding still has its place for binding simple literals like an input's value attribute to a string prop of a value but binding object to object causes pain.  I believe in ember 2.0, binding will be one way only by default and this is the right move.  You can stop this now by ensuring you use ```Ember.computed.oneWay``` instead of ```Ember.computed.alias```.
{% codeblock alias.js %}
// good
prop: Ember.computed.oneWay('attr', function() {
}

//bad
prop: Ember.computed.alias('attr', function() {
}
{% endcodeblock %}

###Update
As <a href="https://twitter.com/cowboyd" target="_blank">Charles Lowell</a> rightly pointed out in the comments, it is possible to create ```readonly``` computed properties that will raise an exception if a ```set``` is an attempted.
{% codeblock readonly.js %}
prop: Ember.computed.readOnly('attr', function() {
}
{% endcodeblock %}
I'll certainly be using this from now on.

One of the benefits of immutability is that diffing is done by value and not by reference, just think how easier diffing javascript would be if the following was true out of the box:
{% codeblock diff.js %}
[1, 2, 3] === [1, 2, 3] //=> sadly false;
{% endcodeblock %}
Libraries like <a href="https://github.com/omcljs/om" target="_blank">om</a> in clojurescript use diffing on their immutable datastructures to great effect to negate needless re-renders with reacts virtual DOM.

But the main reason that I am starting to see the main use case for immutability is that, it makes your code much easier to reason about and you can avoid *side effects* using immutable data structures.  For me, the biggest thing about using immutable data structures is that I am completely safe from some unkown mutation happening which in a long running single page application is great for my peace of mind.  I've been surprised too many times in the past.

You don't have to use a library like <a href="https://github.com/rtfeldman/seamless-immutable" target="_blank">seamless-immutable</a> but you should try and tranfrom or create new objects or arrays rather than mutating existing objects.  You should favour ```map``` or ```filter``` over ```forEach```, ```forEach``` must have a side effect somewhere to accomplish anything.  The only time I would use ```forEach``` is for something like this:
{% codeblock forEach.js%}
arr.forEach(function (item) { console.log(item); });
{% endcodeblock %}

####Statelessness
Imperative programming works by changing programming state to produce answers.  Conversely, functional programming produces answers through stateless operations.

A good example of the contrast is looping, an imperative approach is to use looping:
{% codeblock loop.js %}
var loop = function() {
    for (var x = 0; x < 10; x += 1) {
        console.log(x);
    }
};
loop();
{% endcodeblock %}
The loop produces its results by constantly changing the value of ```x```.  A functional approach would use recursion:
{% codeblock recursion.js %}
var loop = function(n) {
    if (n > 9) {
        console.log(n);
        return;
    } else {
        console.log(n);
        loop(n + 1);
    }
};
loop(0);
{% endcodeblock %}

Functional programming requires us to write stateless expressions and keep our data immutable.

###Back to The Real World......and Ember
If you perform a google for *functional javascript* you will get plenty of examples of sorting lists or simple number crunching.  I have found very little about how to use functional techniques for the type of situations you face on a day to day basis as a web dev or your ongoing battle with the DOM.  So I am going to use a real world example that I worked on within the last week of writing this blog post.

The last thing I worked on was a custom query builder where the user builds a query on the fly from the html elements he is presented with.  Query items can be added and removed:
{% img /images/filter2.png %}
If I had attempted something like this 12 months ago, I would have used rampant mutation, observers and two way binding in a huge chaotic side effect soup.  I have my battle scars and I see the world differently.  I will now detail my solution.  I have created this <a href="http://jsbin.com/nesomi/47/edit?html,js,output" target="_blank">jsbin</a> that you can play along at home with and is a pretty damn accurate duplicate of how I solved this.

The truth of the matter is that ember is very, very, very stateful and adheres more to a class or object based methodology.  If you want to reference data structures in your templates then you need to create member properties and reference them.

My goal for any new piece of code that I create is to keep things as side effect free as possible.  I will most definitely never use the ```Ember.observer``` primitive, I use few computed properties if any and rarely use two way databinding.  I will work with immutable copies of anything persistable.  I want to keep surprises to a minimum and I want to keep things in my control and not be working or waiting for the run loop which is outside of my control.

In this custom query example, there is only ever one active query that you are manipulating, you are either building a new one or you are updating an existing query.  These queries are persistable and I have modelled it as a parent query that contains a child collection of *query parts*.  Here is the ember data *CustomQuery* definition:
{% codeblock CustomQuery.js %}
App.CustomQuery = DS.Model.extend({
  name: DS.attr('string'),
  queryParts: DS.hasMany('queryPart', {async: true})
});
{% endcodeblock %}

And here is the *queryPart*:
{% codeblock queryPart.js %}
App.QueryPart = DS.Model.extend({
  field: DS.attr('string'),
  operator: DS.attr('string'),
  value: DS.attr('string')
});
{% endcodeblock %}
My main goal for working with instances of these ember-data models is to not work with them.  I am going to create copies of the data structure and work with those and only copy them back into the models when I want to persist anything.  The last thing I am going to do is to use two way databinding to bind directly to the model properties.  I have been down that path and it is the road to hell.

At any one time, we are dealing with a collection of query parts and we have *query-builder* component to add and remove query parts.  Below is a screenshot of how the elements look:
{% img /images/query-builder.png %}

The ```IndexRoute``` in the jsbin, either creates an empty array and assigns it to a ```queryParts``` variable that is passed to a ```query-builder``` component, or if we are dealing with a persisted query from the models above then we create a raw hash to pass to the component:
{% codeblock indexroute.js %}
App.IndexRoute = Ember.Route.extend({
  model: function(params, transition){
    var queryParts = {};

    if(params.query) {
      var query = this.store.all('customQuery').find(function(c){
        return c.get('id') === params.query;
      });

      queryParts = query.get('queryParts').toArray().map(function(part){
        return {
          field: part.get('field'),
          operator: part.get('operator'),
          value: part.get('value')
        };
      });

      this.controller.set('queryParts', Ember.A(queryParts));
    } else {
      this.controllerFor('index').set('queryParts', Ember.A());
    }

    return Ember.RSVP.hash({
      contacts: this.store.find('contact', {query: queryParts}),
      customQueries: this.store.find('customQuery')
    });
  },
{% endcodeblock %}
The user can navigate to the ```IndexRoute``` via a link that will have the custom query id in the query params that you can see on ```line 5``` of the above code.  I am creating a hash rather than using the ember-data model directly to keep things side effect free.  I have already made my life much easier.

Here is the how *query-builder* component is referenced on the parent template or the *index* template in the <a href="http://jsbin.com/nesomi/47/edit?html,js,output" target="_blank">jsbin</a>.  The binding of the ```queryParts``` that are passed into ```query-builder``` are sadly still two way bound at this time of writing with ```ember 1.11.1``` but we will work with them as if they were one way bound.
{% gist c8894c38662d15138ea5 %}
Below is the template for ```query-part.hbs```:
{% gist 063829fab7b3f7c0491f %}
On lines ```2 to 4```, a ```query-part``` component is rendered for each part of the query.  The template for ```query-part.hbs``` is below:
{% gist e67ac51fbcbccee34e24 %}
The ```query-part``` component will fire off queries in response to either user input in the textbox or changing the query part operator in the dropdown.

Below is the code for the ```query-part``` component
{% codeblock query-part.js %}
App.QueryPartComponent = Ember.Component.extend({
  actions: {
    changeOperator: function() {
      this.sendQuery();

      return false;
    },
    removeQueryPart: function(index) {
      this.sendAction('removeQueryPart', this.get('index'));
    }
  },
  sendQuery: function() {
    Ember.run.debounce(this, 'executeQuery', 300);
  },
  executeQuery: function() {
    var self = this;

    Ember.run.next(function() {
      if(!self.$('input[type=text]')) {
        return;
      }

      var q = Immutable(self.get('queryPart')),
          value = self.$('input[type=text]').val() || '',
          operator = self.$('.operatorSelector').val(),
          index = self.get('index');

      if(!value.length) {
        return;
      }

      self.sendAction('modifyQuery', q.merge({operator: operator, value: value}), index);
    });
  },
  keyDown: function(e) {
    if(e.keyCode === 13) {
      return false;
    }

    this.sendQuery();
  },
{% endcodeblock %}
We want to adhere to data down and actions up.  So queries are bubbled up from this component to the controller.

If input is entered into the text input, the ```keyDown``` handler on ```line 35``` calls ```sendQuery``` on ```line 12``` that will ```debounce``` a call to ```executeQuery```.

The code in ```executeQuery``` on lines ```15-34```, could definitely be accused of being more jQuery than ember.  If that is the case, then I am guilty as charged.  I am using simple selectors to extract the values from the element rather than relying on two way databinding.  I don't want any surprises about values not being sync'd, I just want to extract the values from the elements when the query is run.  There really is no need for it to be any more complicated than that.

I then bubble the newly constructred immutable data structure up to the parent component by calling the ```modifyQuery``` action.  I don't necessarily need these structures to be immutable but I want to guard against any side effects I might encounter unexpectedly.  This action is then bubbled up to the controller where a it uses the raw hash to execute a ```findQuery``` on the main data that the query is filtering:
{% codeblock runcustomQuery.js %}
runQuery: function(queryParts) {
  var self = this;

  this.store.find("contact", {query: queryParts})
    .then(function(results){
      self.set('contacts', results);
    });
  }
{% endcodeblock %}

When it comes to saving the query, the ```query-builder``` component calls a ```saveCustomQuery``` action on the parent controller, passing the raw array of query parts which are plain old javascript objects and either creates a brand new ```CustomQuery``` model and persists it or overrides the existing ```queryParts``` collection of the existing instance:
{% codeblock saveQuery.js %}
saveCustomQuery: function(){
  var self = this,
      all = this.store.all('customQuery').toArray(),
          customQuery;

  if(!this.query) {
    var queryName = "Query " + (all.length + 1).toString();

    customQuery = this.store.createRecord("customQuery", {name: queryName});
  } else {
    customQuery = all.find(function(c) {
      return c.get('id') === self.query;
    });

    customQuery.get('queryParts').clear();
  }

  this.get('queryParts').forEach(function(part){
    customQuery.get('queryParts').createRecord(part);
  });

  customQuery.save().then(function(result){
    self.transitionToRoute('index', {queryParams: {query: result.get('id')}});
  });
}
{% endcodeblock %}
There are only 2 parts in this piece of functionality where I deal with the actual ember-data instances.  The first is when I am creating copies of the structures to feed to the components and here when I am persisting the changes back to the server.

###Epilogue
Congratulations if you made it this far, this is a long post.  Staying side effect free and dealing with pure functions is next to impossible when working with something as stateful as the DOM.  Ember also is very stateful and class based.  I've used a real world example because there is no better example to illustrate just how challenging it is.  I have 100% failed on the pure function front.  But I've tried to stay as side effect free which for me means dealing with copies of anything persistable, avoiding observers, computed properties and two way data binding wherever possible.  I've tried the other way and it is not pretty.  The code I've described will give me a lot less headaches than other ways I might approach this.

I'm interested to see if we can take a much more functional approach with ember 2.0 but the rfc that wants to pollute handlebars helpers with state and life cycle events is not a step in the right direction.

I've heard of the benefits of functional programming for years but I now realise how sensible they are and how they lead to more robust code.  When working in the chaotic world of the single page application, they make nothing but sense.

I would love any feedback at all on this.  Good or bad, state your case.
