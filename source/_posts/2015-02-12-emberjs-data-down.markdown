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

While a lot of this is down to the fragility and unforgiving nature of those early ember-data statemanager, the problem remains that it should not be the responsibility of the downstream ```input``` component to directly mutate the upstream model field and unexpected side effects occurred because of this. The problem is not particularly bad in this example but it becomes a big problem when you have a tree of nested components and shared data amongst these components, then it becomes difficult or sometimes impossilbe to reason about or track down what actually mutated the state. A component has its data changed in some other component that shares the same state, and it has no idea where the mutation occurred.  This had led me to really doubt the viability of ember as something I wanted to use or promote in my day job.  I looked into react and I seen a much cleaner, saner model.  It seems that the ember core team have also been learning from react.

###The Ghost of Christams Future
Thankfully with the announcement of the ember 2.0 <a href="https://github.com/emberjs/rfcs/pull/15" target="_blank">roadmap</a>, the core team appear to recognise the dangers of this rampant mutation and they want to move a way from two way databinding, at least by default anyway.  Databinding will be one way by default in ember 2.0 which greatly simplifies the interaction.  The new paradigm that is being promoted goes by the meme  **data down and actions up**.  Child components will get their data from parent components, so when the parent component changes, the child component's UI is updated.  Data will flow down to the child components and the child components will notify their parent components of data changes by calling actions or pushing data up.  The parent controller should have the responsibility of persisting the changes and pushing those changes to the children.  Now I fully support the data down approach, this makes absolute sense to me but I think that there are still problems with the data up approach, more on this later.  But firstLet me now flesh out the data down, actions up approach with a refactor of the original example in this <a href="" target="_blank">jsbin</a> and below is the updated ```index.hbs``` template:
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
The main thing to take away from the above code sample is that I am passing the parent component;s ```isFinishedChangedhandler``` to the child ```todo-item``` component.  There is new syntax planned for actions but at the time of writing, I am using ember 1.10 and this remains the only way of passing handlers down.
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
