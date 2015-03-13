---
layout: post
title: "Emberjs, reflux, immutability, no ember data, no models....sanity?"
date: 2015-03-13 08:34:15 +0000
comments: true
categories: Ember, JavaScript
---
###Where we are
In my last [post](http://www.thesoftwaresimpleton.com/blog/2015/02/12/emberjs-data-down/), I put forth the argument that the new data flow story of ember next does not go far enough.  I now want to suggest a way that we could bring some much needed sanity to our ember applications.  I have been on the same ember project for 2+ years and here are some of the problems that I have found with ember during those 2+ years:

- Ember often seems opaque and disjointed with a lot of magic happening behind the scenes to piece things together.

- I find the whole bindings, observables and computed properties paradigms hugely problematic.  I find it impossible to hold a mental model in my head about how my properties and dependant keys fit together.  Using observers and computed properties often causes surprises as a whole host of side effects could happen in both the UI and the data flow of the application.  Cascading updates of partially resolved data could happen.  Observers and cps can cause chain reactions of other observers and cps to fire.  I would go as far as to say that both observers and cps should be depreceated.

- I found ember-data unusable, I ended up forking my own version of ember-data ```0.14``` because I was sick of the carpet being pulled from under me with every release.  I patched up the code as best I could and we still use this  bastardised version to this day but it is far from ideal.  Serializing and deserializing models is hugely expensive and does not scale.  I want to work with POJOs not these complex resource hoggers.

###Where we are heading
Data Down and actions up is the new meme being pumped with ember 2.0 but as I mentioned in my last [post](http://www.thesoftwaresimpleton.com/blog/2015/02/12/emberjs-data-down/), this does not go far enough.  Now I fully agree with the *data down* part but what I disagree with is *actions up* or more specifically actions being propagated to a parent component.

The problem with this is that a component is not the right place to change application state.  What you will end up with is application state changing logic broken up and scattered across the whole application in all these various parent components.  The component is not the right place to be dealing with this logic.  Probably the main problem with the component handling application state change is that we will get unwanted or surprising UI side effects, this will be compounded if we are start using cps.  What is needed is a layer above the components that is charged with changing application state.  This layer then notifies subscribers of any changes to application state.  This is described by flux or reflux as a ```unidirectional dataflow```.  I will shortly flesh this out with an example.

###Where we could and probably should go
I have created <a href="http://boiling-waters-4657.herokuapp.com/" target="_new">this</a> TodoMVC style application to illustrate the example and you can find the source code for this application <a href="https://github.com/dagda1/ember-reflux" target="_new">here</a>.  I've added ```undo``` and ```redo``` functionality to illustrate how much easier this is with immutable datastructures which we will get to later.
####Reflux/flux

{% img /images/lasttry.png %}

The basic premise behind flux or reflux is:

- There are actions
- There are data stores (don't think ember-data store)
- The dataflow is unidirectional or data flows down.

The basic premise is that components call actions that initiate new data changes in the data stores (don't confuse these stores with the ember-data store).  Mutation of data can only happen through calling the actions.  The data stores listen on the actions and mutate the data within the store.  This keeps the data structure flat and lets the concerns of data manipulation be the purpose of the store.  This totally removes the chance of side effects that may happen if components are charged with handling data on their own.

The stores then let the components know of any new data changes as a result of the actions.  When data flows in a single direction like this, it is much easier to follow or reason about what is happening in your application which is in contrast to the state of ember today.  Components can only manipulate or mutate data by executing an action.

If we take the example of the <a href="http://boiling-waters-4657.herokuapp.com/" target="_new">todo application</a>, I have a file where I add any new ```actions``` that will be needed to manipulate the todos:
{% codeblock actions.js %}
export default [
  "addItem",
  "toggleItem",
  "toggleAll",
  "getInitial",
  "getTodos",
  "undo",
  "redo",
  "editItem",
  "destroyItem"
];
{% endcodeblock %}

I will use creating a todo to illustrate the data flow.  The above code lists an array of ```actions``` that signify all the operations that were required to manipulate the todos in the application and on ```line 2``` is an ```addItem``` action that a component would call to create a todo.  I hacked together a basic pubsub mechanism to mimic the role of the dispatcher in the real reflux library.  This pubsub mechanism or poor man's dispatcher is used to raise events from the components to the stores that are subscribed to these actions.  I used an <a href="https://github.com/dagda1/ember-reflux/blob/master/app/initializers/init-actions.js#L5" target="_new">initializer</a> to read the array of actions above.  A ```TodoActions``` object is created and injected into all components.  This will decouple the component and the store.

Manipulating todos is done through this UI:

{% img /images/todos.png %}

The various components are arranged like this:

{% codeblock %}
<section id="todoapp">
  <div>
    &#123;&#123;todo-header&#125;&#125;
    &#123;&#123;todo-list&#125;&#125;
    &#123;&#123;todo-footer&#125;&#125;
  </div>
</section>
{% endcodeblock %}

The ```todo-list``` component in turn creates ```todo-item``` components like so:

{% codeblock %}
<section id="main">
  <input &#123;&#123;action "toggleAll"&#125;&#125; id="toggle-all" type="checkbox">
  <ul id="todo-list">
    &#123;&#123;#each todo in todos&#125;&#125;
      &#123;&#123;todo-item todo=todo tagName="li"&#125;&#125;
    &#123;&#123;/each&#125;&#125;
  </ul>
</section>
{% endcodeblock %}

The ```TodoHeader``` component contains the textbox that listens for ```keyDown``` events and if enter is pressed it will call the ```addItem``` action on ```line 6``` of the code listing below:

{% codeblock header.js %}
keyDown: function (e) {
  Ember.run.next(() => {
    let text = e.target.value || '';

    if(e.which === 13 && text.length) {
      this.TodoActions.addItem(text);
      e.target.value = '';
      e.target.focus();
    }
    else if (e.which === 27) {
      e.target.value = '';
    }
    });
},
{% endcodeblock %}

The pubsub mechanism will then let any subscribed stores know that an ```addItem``` event has occurred.

I have created a ```TodoStore``` object and included this simple mixin in the ```TodoStore``` to subscripe to the array of actions:
{% codeblock connect.js %}
import Ember from 'ember';

export default Ember.Mixin.create({
  connect: Ember.on('init', function(){
    this.Reflux.connectListeners.call(this, this.listenables);
  })
});
{% endcodeblock %}
The mixin is then included in the ```TodoStore``` definition where it pulls in the list of actions via the ```listenables``` property.

{% codeblock store.js %}
export default Ember.Service.extend(ConnectListenersMixin, {
  listenables: TodoActions,
{% endcodeblock %}

The convention of reflux is to name your handler ```onAction``` and the ```ConnectListenersmixin``` will check for the correct handlers on the class that includes it and subscribe the store to the array of actions that was listed earlier.

When ```TodoActions.addItem``` is called from the store, the pubsub mechanism will try and route the event to an ```onAddItem``` handler in the store.

The ```TodoStore```, ```onAddItem``` handler is listed below:
{% codeblock addItem.js %}
onAddItem: function(description) {
  var newItem = {
    key: UUID4.generate(),
    created: new Date(),
    finished: false,
    description: description
  };

  let newList = mori.conj(this.todos, newItem);

  this.undoList = mori.cons(this.todos, this.undoList);

  this.updateList(newList);
},
{% endcodeblock %}

The application has undo and redo functionality and I've used immutable data structures to make this fairly trivial.  I've used <a href="https://github.com/swannodette/mori" target="_blank">mori's</a> persistable data structures because I am familiar with the data structures api from hacking around on clojurescript.  Using immutable data structures opens up many doors such as easier comparison checking, diff checking and memoization to name but a few.

Immutable data structures are called *persistent datastructures* because they *persist* a previous version of themselves when modified.  This is ideal for redo and todo functionality.  I simply add the previous version to an ```undoList``` vector whenever the ```todos``` vector changes on ```line 11``` of the code above.  I have ```undoList``` and ```redoList``` vectors and I simply ```cons``` the application state onto the relevant vector that I can pop off when they are called....via actions of course!

The call to ```mori.conj``` will create a new persistent datastructure on ```line 9``` of the above code and I then call ```updateList``` that looks like this and will then *send* the data down to any components that are listening for store events.  ```updateList``` looks like this:

{% codeblock updateList.js %}
updateList: function(newList) {
  let payload = this.getPayLoad(newList);

  localStorage.setItem(localStorageKey, JSON.stringify(mori.toJs(newList)));

  this.trigger('listUpdated', payload);

  this.todos = newList;
},
{% endcodeblock %}

The ```updateList``` function calls ```getPayLoad``` that packages up the updated list for the components to rerender based on the changes:

{% codeblock getPayload.js %}
getPayLoad: function(list) {
  let filter = get(this, 'filter'),
      todos = null,
      todosLeft = mori.count(mori.filter(filters['active'], list));

  todos = mori.filter(filters[filter], list);

  return {
    todos: mori.toJs(todos),
    canRedo: mori.count(this.redoList) > 0,
    canUndo: mori.count(this.undoList) > 0,
    todosLeft: todosLeft
  };
},
{% endcodeblock %}

```getPayLoad``` returns the current list of todos and also some additional information that is used to render the various components.  Using this technique, I can totally avoid the computed properties that I would normally create for this type of operation.  The components know nothing about the internal structure of the todos or that they are immutable data structures in the store as they are converted to normal POJOs on ```line 9``` of the above code block.

I then include the following mixin in all the relevant components that want to listen to store events and re-render the application state changes:

{% codeblock listener.js %}
import Ember from 'ember';
import ConnectListenersMixin from '../mixins/connect-listeners-mixin.js';

let set = Ember.set;

export default Ember.Mixin.create( ConnectListenersMixin, {
  listenables: ['listUpdated'],

  onListUpdated: function(payload){
    set(this, 'todos', payload.todos);
    set(this, 'canUndo', payload.canUndo);
    set(this, 'canRedo', payload.canRedo);
    set(this, 'todosLeft', payload.todosLeft);
  }
});
{% endcodeblock %}

The store will not push these updates until all application state changing code has executed.  I won't get side effects or other surprises until the store has finished its work and pushed the changed application state back down.

The upshot of this architecture is that it is very easy to reason about, all mutations are happening in a controlled and logical fashion.  I have not used a computed property nor have I used an ember-data class but of course this example is very trivial.

All the actions take a similar flow to this, below is the ```onEditItem``` ```TodoStore``` handler that has to clone the todo first to before making state changes:

{% codeblock editItem.js %}
onEditItem: function(key, description) {
  let existing = getExistingByKey(key, this.todos),
      index = getIndexByKey(key, this.todos),
      cloned = $.extend({}, existing);

  this.undoList = mori.cons(this.todos, this.undoList);

  cloned.description = description;

  var newList = mori.assoc(this.todos, index, cloned);

  this.updateList(newList);
}
{% endcodeblock %}

Undo/redo is simple in this case as we are not having to roll back the changes on the server but diff checking is considerably easier with immutable data structures and we could use that to see what has changed.  Below is how the ```undo``` action looks:

{% codeblock undo.js %}
onUndo: function() {
  let reverted = mori.first(this.undoList);

  this.redoList = mori.cons(this.todos, this.redoList);
  this.undoList = mori.drop(1, this.undoList);

  this.updateList(reverted);
},
{% endcodeblock %}

### Conclusion
I'm too long in the tooth to get too carried away with this simple example, I have no Ajax calls, web sockets or even nested data structure so the jury is still out on how this would pan out in a real application.  Another question would be, why not just use react?  I would use react if I was not on my current project but I am not about to advocate refactoring 2+ years of code to a different MVC framework.

What I like about what I have discovered with this simple experiment is:

- I have not used any computed properties.
- The data flow is easy to follow and I know where mutation happens.
- I have not needed any ember data or model classes.  After hacking on clojurescript and clojure, I honestly don't think an expressive model is that important to represent state.  I want to see how far I can go with POJOs and arrays.
- Immutable data structures opens up many exciting possibilities.
- I have an adequate mental model of what is happening, I am less likely to get surprised by what is happening or the often wild west nature of an ember application.

If you have any feedback on any of this then leave a comment below.
