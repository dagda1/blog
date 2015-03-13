---
layout: post
title: "ember.js - Data down, actions up...better but still not far enough"
date: 2015-02-12 17:10:46 +0000
comments: true
categories: Ember JavaScript
---
###The Ghost of Christmas Present
It is ironic that the thing that first drew me to ember is the thing that ended up causing the most excruciating pain.  I am of course talking about two way data binding.  An example of this is in this <a href="http://jsbin.com/pitupo/1/edit?html,js,output" target="_blank">jsbin</a> and in the code below which renders a list of todos.  Two way databinding is employed to synchronise the ```isFinished``` boolean property of the todo and with the ```checked``` property of the input component:
{% codeblock %}
&#123;&#123;&#35;each todo in active&#125;&#125;
  <tr>
    <td>&#123;&#123;input type="checkbox" name="isFinished" checked=todo.isFinished&#125;&#125;</td>
    <td>&#123;&#123;todo.description&#125;&#125;</td>
  </tr>
&#123;&#123;/eachn&#125;&#125;
{% endcodeblock %}
After years of DOM fiddling with jquery, this was instantly appealing as changes to the DOM are synchronised with the bound model without any extra code on from the developer.  While this was great for breath taking ember demos, it led to a pandora's box full of problems in produciton with earlier version of ember-data.  In the first versions of ember-data, binding a model in this way led to the model's ```isDirty``` flag getting set and if the model was not saved or the changes were not rolled back, the model remained in this dirty state and multiple errors would occurr as undefined methods were called on the current dirty state.  I hacked around this pain by using the <a href="https://github.com/yapplabs/ember-buffered-proxy" target="_blank">buffered proxy</a> and other outlandish hacks but dealing with this is not something I ever want to experience again and still causes me to scream out in my sleep which is distressing for my partner.

While a lot of this is down to the fragility, rigidity and unforgiving nature of the early ember-data statemanager, the problem remains that it should not be the responsibility of the downstream ```input``` component to directly mutate the upstream model field and unexpected side effects occurred because of this. The problem is not particularly bad in this example but it becomes a big problem when you have a tree of nested components and shared data amongst these components, then it becomes difficult or sometimes impossible to reason about or track down what actually mutated the state. A component has its data changed in some other component that shares the same state has no idea where the mutation occurred.  Ember needs a data flow story that is currently missing and computed properties and observables make the flow of data in an ember application very difficult to reason about and cascading partial updates from partially resolved asynchronous requests can get very stressful, I'll elaborate more on this later but the ember core team seems to have reacted (pun intended) to some of the good things that react are doing and one way databinding will be the default as of ember 2.0.

###The Ghost of Christams Future
Thankfully with the announcement of the ember 2.0 <a href="https://github.com/emberjs/rfcs/pull/15" target="_blank">roadmap</a>, the core team appear to recognise the dangers of this rampant mutation and they want to move a way from two way databinding, at least by default anyway.  Databinding will be one way by default in ember 2.0 which greatly simplifies the interaction.  The new paradigm that is being promoted goes by the meme  **data down and actions up**.  Child components will get their data from parent components, so when the parent component changes, the child component's UI is updated.  Data will flow down to the child components and the child components will notify their parent components of data changes by calling actions or pushing data up.  The parent component should have the responsibility of persisting the changes and pushing those changes to the children.  Now I fully support the data down approach, this makes absolute sense to me but I think that there are still problems with the data up approach, more on this later.  But first
let me now flesh out the data down, actions up approach with a refactor of the original example in this <a href="http://jsbin.com/pitupo/2/edit?html,js,output" target="_blank">jsbin</a> and below is the updated ```index.hbs``` template:
{% codeblock %}
<h1>Todos</h1>
&#123;&#123;todo-list todos=model&#125;&#125;
{% endcodeblock %}
It seems that the proxying controllers ```ArrayController``` and ```ObjectController``` are going to be depreceated and components will be the main currency so my ```todo-list``` becomes the top level component and below is its template:
{% codeblock %}
<table>
  <tbody>
    &#123;&#123;#each todo in active&#125;&#125;
    <tr>
      &#123;&#123;todo-item todo=todo isFinishedChanged="isFinishedChangedHandler" tagName="td" &#125;&#125;
    </tr>
    &#123;&#123;/each&#125;&#125;
  </tbody>
</table>
{% endcodeblock %}
The main thing to take away from the above code sample is that I am passing the parent component's ```isFinishedChangedhandler``` to the child ```todo-item``` component.  There is new syntax planned for actions but at the time of writing, I am using ember 1.10 and this remains the only way of passing handlers down.
Below is the component's code:
{% codeblock todo-list.js %}
App.TodoListComponent = Ember.Component.extend({
  actions: {
    isFinishedChangedHandler: function(todo) {
      todo.toggleProperty('isFinished');
      todo.save();
    }
  },
  active: Ember.computed.filterBy('todos', 'isFinished', false)
});
{% endcodeblock %}
The ```todo-item``` child component that is rendered from the parent ```todo-list``` component has the following template:
{% codeblock %}
  <input type="checkbox" name="todo"/> &#123;&#123;todo.description&#125;&#125;
{% endcodeblock %}
Gone is the ```input``` component and the bound ```isFinished``` property with the ```input's``` ```checked``` property.

Instead the ```isFinishedChangedHandler``` is called by the downstream component in the change event:
{% codeblock todo-item.js %}
App.TodoItemComponent = Ember.Component.extend({
  change: function(e) {
    this.sendAction('isFinishedChanged', this.get('todo'));
    return false;
  }
});
{% endcodeblock %}
I've used ```sendAction``` but I expect this to change in ember 2.0.  This is nice and neat in this simple of simplest examples.  Once a checkbox is clicked the ```isFinishedChangedHandler``` of the parent controller is called and any state changes are synchronised down to the child components via the one way binding.

Ok great, I buy into this with this example but what if we have a more involved form with a number of text input fields, are we saying that we should write handlers for each input?  This would very quickly get verbose, in react land, they have the <a href="http://facebook.github.io/react/docs/two-way-binding-helpers.html" target="_blank">ReactLink</a> and I think in ember they are proposing something along the lines of this:
{% codeblock %}
<input value=&#123;&#123;mut firstName&#125;&#125;>
<input value=&#123;&#123;mut lastName&#125;&#125;>
{% endcodeblock %}

###Data Flow in a Real world application
I see a lot of hype about html syntax for components, ok my templates look more like html but what I want from ember is a real dataflow story.  React has covered this with <a href="http://facebook.github.io/flux/docs/todo-list.html" target="_blank">flux</a> and even better again with <a href="https://github.com/spoike/refluxjs" target="_blank">reflux.js</a> where the data flow is unidirectional.  In the real world application that I work on, I had a reasonably complex data table where all fields in the table needed to be editable in various ways and I was dealing with a large dataset, the table is infinitely scrolled and every sort operation etc. goes to the server.  The sort macro etc. are useless to me in this scenario because all the objects are not in memory.  There is a textbox on the top right which will filter the records further by whatever search term is entered.  Below is how the table looks:
{% img /images/table.png %}
Now where ember really shone was allowing me to add components to each table cell with a little bit of hacking that I blogged about <a href="http://www.thesoftwaresimpleton.com/blog/2014/11/18/dynamic-content/" target="_blank">here</a>.  This was great, every table cell is editable with a little help from html5's <a href="https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Content_Editable" target="_blank">contenteditable</a>.  The backing dataset of this table can have 1000s of records and there are multiple filters and sorting operations that all cause a requery of the external asynchronous data source.  How does data come in and out of the the ember application?  Currently ember lacks a data flow story which is disappointing in my opinion.  How do I orchestrate the data coming from the server with its many filters and sort properties are selected from user input?

So the way I got round this was to use a combination a combination of <a href="http://emberjs.com/guides/routing/query-params/">query params</a>, computed properties and observables.  When the user selects a filter from the left menu in the screenshot above, a ```transitionTo``` action will occurr to a url with the filter added to the query params, something like ```addressbook/people/tagged?tag=191&company=856```, this will cause execution to return to the route and select the new data.  What do we do if a column is sorted or what do we do if some search text is entered to further filter the data?  My solution is to maintain through a computed property the current parameters that will be sent to the server:
{% codeblock filterparams.js %}
filterParams: Ember.computed('filter', 'user', 'tag', 'company', function() {
    var company, contactimportjob, params, tag, user;
    params = {
      filter: this.get('filter'),
      page_size: this.get('pageSize')
    };

    params.company = this.get('company');
    params.user = this.get('user');
    params.tag = this.get('tag');

    return params;
  })
{% endcodeblock %}

I do something similar with the sort which triggers a query param change that goes back to the route to fetch the new data.  For the search I use an observer that listens for a bound property change:

{% codeblock search.js %}
  searchDidChange: Ember.observer("searchText", function() {
    var filterParams, params, searchText;
    if (this.get('filter') === null) {
      return;
    }
    searchText = this.get('searchText');
    filterParams = this.get('filterParams') || {};
    params = Ember.merge(filterParams, {
      like: searchText
    });
    return Ember.run.debounce(this, 'contentChanged', Ember.copy(params), 150);
  })
{% endcodeblock %}

I could have used query params for this but I did not, I am also having to debounce as the input changes which is less than ideal.  The solution I have come up with feels disjointed and I had to remind myself of how it all fits together when I write this.  I am also dealing with partial updates happening from data coming in that is not fully resolved from the asynchronous source.  You end up putting conditional logic or using inline observers that can really get quite annoying.

The problem with the above is that changes happen with responses to computed properties and observables that cause state mutation in a very disjointed fashion.  With something like flux every user initiated change in the UI is communicated to the dispatcher via an action.  The same goes for any data coming from a REST endpoint,  where an action can be fired to the dispatcher. This is a top down approach that is consistent across the application and I'm not rolling out new ways of tackling problems for each scenario. This is a top down orchestrated workflow and not side effect driven like we have with ember and computed properties.  I've really lost faith in the computed property and observable paradigm which causes no end of problems with asynchronous data.  Ember seems to put more focus on quirky little macros that work on small datasets or nice html syntax for htmlbars which really does not make my life much easier.  I can live with ```bind-attr``` for now, but I ended up forking and using a bastardised version of ember-data because I was sick of replacing one set of problems for another on every new release.  Why is ember-data not the focus, I would love to know.  Please feel free to shout me down for this last sentence as this really is just my opinion.

So to sum up, I feel that ember is at least making a move in the right direction with data down, action up but more needs to be done.  One way databinding by default is also the right move.  But we need a dataflow story and we need to forget about things like nicer html syntax or at least forget about the flashy demo stuff and concentrate on the problems that user's have with real data on real applications.  The fact that ember-data is still not 1.0 is just insane.  This gives weight to my argument that ember often lets the more mundane important stuff take a back seat to the flashy nicer stuff, e.g. html syntax for components.  Ember has some of the best developers out there working on it so they really can and should address these problems.

With all this said, the application I work on is in production and we have paying users but a lot of time was spent on issues with ember and ember-data rather than creating features.  Ember is free and it has helped us in a lot of ways and I find it really useful for a lot of things like creating widgets.  I can't remember the last time I downloaded a jquery plugin.  The router is also a thing of great beauty and something that really shines.  Ember-cli is also a great tool also.  I hope my criticism is constructive and not bitchy as I have a lot of respect for the developers.

React seems to be moving to fully embrace immutable data structures and I find this very exciting.  Immutability opens up a new world of possiblities and makes things like change tracking, memoization and undo/redo functionality trivial.
