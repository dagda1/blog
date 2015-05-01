---
layout: post
title: "Emberjs - Action Maker"
date: 2015-05-01 17:25:23 +0100
comments: true
categories: Ember, JavaScript
---
I am currently working on an ember project using ```1.7.1``` and I've reached the point of no return with regard to passing action names down a component hierarchy and then bubbling them back up using ```sendAction```.  I believe the problem is still present in recent versions of ember and we need to somehow make this more user friendly.

I will illustrate the problem with an easily grokked todo example before giving a couple of solutions.

Here is a list of todos:
{% gist f2c0e3f45ed313dbd9c4 %}

Here I am creating a ```todo-list``` component and specifying an action that I want called when a todo is checked to signify it is finished.

Below is a route with a ```finishTodo``` action that is referenced in the above code sample:
{% codeblock daddy.js %}
App.IndexRoute = Ember.Route.extend({
  actions: {
    finishTodo: function(todo) {
      console.log('in root');
      todo.set('isFinished', true);
      todo.save().then(function(){
        console.log('finished');
      });
    }
  },
  model: function(){
    return this.get('store').find('todo');
  }
});
{% endcodeblock %}

All good so far but with the current state of **data down and actions up**, I need to pass this action name down to each child component that would use ```sendAction``` to call it.

In the example, I need to specify the action name in each ```todo-item``` component that the ```todo-list``` component creates:
{% gist 73487ea0117c2f1b25f7 %}

I then need to bubble this up from the child ```todo-item``` component:
{% codeblock todo-item.js %}
App.TodoItemComponent = Ember.Component.extend({
  actions: {
    finishTodo: function(todo) {
      console.log('in child');

      this.sendAction("action", todo);
    }
  }
});
{% endcodeblock %}

This gets bubbled up to the ```todo-list``` component that uses ```sendAction``` to bubble this up to the route:

{% codeblock todo-list.js %}
App.TodoListComponent = Ember.Component.extend({
  actions: {
    finishTodo: function(todo) {
      console.log('finish in parent component');

      this.sendAction("action", todo);
    }
  }
});
{% endcodeblock %}
Here is a <a href="http://jsbin.com/pepule/1/edit?html,js,output" target="_blank">jsbin</a> that shows the up and down duplicity in its entirety.

One sure sign of tight coupling is the number of places/files that you need to change for one method name change.  As developers, we should be seeing this type of thing as a red flag that requires attention.

One way round this is to create a helper to package up the action, the context and any arguments that are needed to call the action.  This can be achieved with a handlebars subexpression and a handlebars helper.

I can refactor the ```todo-list``` component to this:
{% gist d14bff03ccbf46de4ca2 %}

The above code uses a handlebars subexpression to create a method named ```action``` on each ```todo-item``` component, I pass in the ```targetObject``` which in this case is the ```controller``` but you can reference anything that is accessible and has an actions hash, I also pass the action name I want called and any additional arguments.

The subexpression calls this handlebars helper that returns a higher order function:
{% codeblock makeAction.js %}
Ember.Handlebars.registerBoundHelper('makeAction', function() {
  var context = arguments[0],
    args = Array.prototype.slice.call(arguments, 1 , -1);

  return function() {
    return context.send.apply(context, args);
  };
});
{% endcodeblock %}

I can then call this action in the ```todo-item``` component:
{% codeblock %}
App.TodoItemComponent = Ember.Component.extend({
  change: function(e) {
    var todo = this.get('todo');
    this.action(todo);
  }
});
{% endcodeblock %}

Here is a <a href="http://jsbin.com/lipeno/4/edit" target="_blank">jsbin</a> of the above.

This makes things a little more sane for me and I can eliminate a lot of code.  Another alternative that I am considering is to enhance the <a href="http://www.thesoftwaresimpleton.com/blog/2015/04/27/event-bus/" target="_new">global event bus</a> I previously blogged about.

The alternative is to carry on with what is currently on offer but I've just reached my limit with that.
