---
layout: post
title: "Ember.js - Model, View, StateMachine?"
date: 2012-02-28 13:11
comments: true
categories: JavaScript Ember
---
I have been using <a href="http://documentcloud.github.com/backbone/" target="_blank">Backbone.js</a> in my day job recently for quite a few months on a thick client javascript application.  I have enjoyed the order and structure that the framework has brought to my javascript code.  I have also learned a lot of organisational and structural techniques by reading the book <a href="http://www.amazon.co.uk/JavaScript-Patterns-Stoyan-Stefanov/dp/0596806752/ref=sr_1_1?ie=UTF8&qid=1330435566&sr=8-1" href="_blank">JavaScript Patterns</a> recently.

Like most developers, I am working on a sure fire billion dollar side <a href="http://www.leadcapturer.com" target="_blank">project</a>.  In order to maintain the clear distinction between my day job and my side project, I have decided to use <a href="http://emberjs.com/">Ember.js</a> as my client side framework for this billion dollar baby.  I have been very impressed by what I have found so far.  I have also found an interesting new paradigm for clientside MV* that inspired me to write this post.  I don't want to say too much about the differences between backbone and Ember or even too much about Ember as there is a wealth of information already out there but I will point out my preferences for using Ember.js over Backbone.js:

- **Bindings -** Ember relies heavily on bindings as the core abstraction while backbone relies on raw DOM events.  A binding simply connects the properties of two objects so that whenever the value of one property changes, the value of the other one changes.  For example below is a simple Ember controller that contains a simple Ember view:
{% gist 1932996 %}
On line 7 is a property called *urlSearchBinding*. Ending a property with **Binding** tells Ember to create a binding between the object defining the binding which in this case is the view and the referenced object on the right hand side of the property definition which in this case is the url_search object.  I can now reference this binding in my handlebars template like this:
{% gist 1933213 %}
On line 2 of the above gist, a binding is created between the value property of the Ember.TextField object and the *search_url* property of the *urlSearch* object that is bound by the urlSearchBinding on the view.  The upshot of this is that whenever the value attribute of the input changes, the urlSearch's search_url property  changes.  I much prefer this approach than having to pull values out of the DOM.  There is a lot more to bindings that you can read about in the <a href="http://ember-docs.herokuapp.com/#doc=Ember.Binding&src=false" target="_blank">Ember Docs</a>.  You can also set up <a href="http://ember-docs.herokuapp.com/#doc=Ember.Observable&src=false" target="_blank">observables</a> that help tie the whole Ember MVC pattern together.

- **Out of the box Handlebars support - ** Ember takes a more opinionated view on which clientside templating library to use and comes already hooked up with <a href="http://www.handlebarsjs.com">Handlebars</a> support.  It is possible to compile your templates serverside on *Rails* by hooking up a Rails initialiser to look something like <a href="https://gist.github.com/1780841" target="_blank">this</a>.  I use <a href="http://documentcloud.github.com/underscore/#template" target="_blank">underscore.js templating</a> with backbone and handlebars is by far the superior library.  I have also used JQuery templating which is my least favourite of the three I have tried.

##OK, but what about StateMachines?##
While looking into Ember I found an interesting Ember addon called the <a href="https://github.com/emberjs-addons/sproutcore-statechart" target="_blank">Ember Statechart</a>.  I have no idea why they called this a statechart and not a statemachine which is really what it is.  The <a href="http://en.wikipedia.org/wiki/Finite-state_machine" targe="_blank">state machine</a> pattern is a tried and trusted way of organising computer programs that has existed for a hell of a long time.  Simply put a state machine can be in only one of a finite number of states.  The machine is only in one state at a time;  the state at any current time is called the *current state*.  It can change from one state to another when initiated by a triggering event or condition, this is called a transition.  The classic example is a state machine that represents a lightbulb transitioning from the off state to the on state.  It is also possible to associate actions to a state, for example:

- **Entry Action - ** which is performed when entering a state.
- **Exit Action -** which is performed when exiting a state.

In the ember statechart, these two actions or handlers manifest themselves as the *enterState* action and the *exitState* action and appear on every state.  These actions or handlers are a perfect fit for complex UI interactions.  A client MV* application is truly stateful and there is always init and dispose code that needs to happen as you transition between views or states. The entry and exit state actions provide a perfect place to initialise the views, persist state, hide and show elements or dispose of views and resources.  I recently coded a complex tabcontrol that could have benefitted greatly from a statemachine to switch or transition between tabs.

##Enough talk, show me some code##
Below is a very simple example of a statechart:
{% gist 1934749 %}
Now let me breakdown the code

- On **line 1** a derived statechart class (for want of a better word) is defined by extending *SC.Statechart*.  The fact that the *Statechart* is namespaced with *SC* and not *Ember* gives hints as to when this addon was written.
- On **line 2** the *monitorIsActive* property can provide you with debug info if enabled.  I have yet to use this facility, so it is currently set to false.
- On **line 4** the *substatesAreConcurrent* property indicates if the state's immediate substates are to be concurrent (orthogonal) to each other. 
- On **line 6** is the *rootState* definition, this is the initial state of the statechart.  All the other states I have defined as orthogonal substates of this *rootState*.
- **Line 7** sets the initialSubstate to *notParsing* for when the instance is instantiated.
- **Lines 9 and 17** define the two substates named *notParsing* and *parsing*.  In my application the user enters a url and the application parses any lead details from the site in question.  At some point the user will submit their request and the application will start parsing the results.
- Below each of these subStates are the *enterState* and *exitState* handlers that in this instance are simply showing and hiding elements or in the case of the *parsing* substate, the execution control is also passed from the statechart to the *parsing_controller*.
- On **line 15** is an additional action on the statechart called *startParsing* that can be called like this:
{%codeblock%}
crawl: =>
  Lead.state_chart.sendAction 'startParsing', @url_search
{%endcodeblock%}
- You can define these actions with states or substates and invoke them via the *sendAction* method and pass up to two arguments.
- On **line 16** I am calling the *gotoState* method of the statechart that transitions between states and invokes the *exitState* action of the current state and *enterState* of the state that is being transitioned to.
- On **line 28** I have a **stopParsing** action that will transition back to the **notParsing** state and trigger the parsing state's exitState action and the notParsing state's enterState action.  This will reset the page and let the user enter another submission.

Below is a test that helped me assert the statechart was working as expected.
{%gist 1944728 %}

##Conclusion##
There is a lot more to the statechart than I have mentioned here and there are some nice touches like the **performAsync** function that you can call when an synchronous action needs to be performed whenever entering a state or exiting a state and **resumeGotoState** that resumes an active goto state process that has been suspended after such an async operation.  I also came across this nice <a href="https://github.com/DominikGuzei/ember-routing-statechart-example" target="_blank">routing example</a> that you might find useful.

I am still totally undecided as to where the statechart fits in.  I think it could take the place of the missing controller in Backbone as it is all to easy to overload your model or view with behaviour and things like a destroy method are still missing on the Backbone.View at this time of writing.  On the Ember side of things where you do have controllers or even the <a href="http://ember-docs.herokuapp.com/#doc=Ember.ArrayController&src=false" target="_blank">Ember.ArrayController</a> which is very useful, I use the statechart to ensure that the view and business logic do not communicate with each other.  I think this separation of concern makes things easier to maintain and stops views, controllers or models from becoming bloated.  

I think the statemachine paradigm is a very nice addition to the current Javascript MV* patterns that are out there.  When you are dealing with MVC on the server, it is not real MVC.  Real MVC is when a model can notify (through the observer pattern) the views about its changes.  Ember facilitates this beautifully through its bindings.  On the server, the controller simply passes the model data to the views that handle the HTML generation which is sent to the browser.  Websockets could potentially change this but their adoption is not quite wide spread yet.  On the clientside we get real MVC and things like an entryState action and exitState action fit beautifully as a way to transition or rearranging your page in response to user interaction.
 
I am not sure if I am advocating MVCS but I really recommend you take a look at this piece of kit because it is definitely food for thought.

Please feel free to comment on anything below.