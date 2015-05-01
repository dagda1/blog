---
layout: post
title: "Emberjs - fillIn Test helper with key events"
date: 2015-05-10 12:39:40 +0100
comments: true
categories: Ember, JavaScript
---
I've imposed a self enforced ban on using observers when using ember.  For example, I previously used two way binding in my template to bind the value attribute of an input element to a property
{% gist 43685698bc0149df9c51 %}

I would then create an observer on this bound value like this:
{% codeblock observer.js %}
queryDidChange: Ember.observer('query', function(){
  //some transofomation
}
{% endcodeblock %}

This has generally led to a world of pain that I lamented about in <a href="http://www.thesoftwaresimpleton.com/blog/2015/04/07/observables-evil/" target=_blank">this post</a>.

What I do now in this situation is use something like the <a href="https://developer.mozilla.org/en-US/docs/Web/Events/input" target="_blank">input DOM event</a>:
{% codeblock input.js %}
  input: function(e) {
    var query = e.target.value || '';

    set(this, 'query', query);
    //do stuff
{% endcodeblock %}
I can do any transformations in this event handler without the pain of obsevers or 2 way binding, I am in control of what is happening.

This is all well and good but what is not good is that this is not testable with the current <a href="http://guides.emberjs.com/v1.10.0/testing/test-helpers/" target="_blank">ember test helpers</a>.

There is the <a href="http://emberjs.com/api/classes/Ember.Test.html#method_fillIn" target="_blank">```fillIn```</a> test helper but this will just set the value of an input to the supplied string without raising any key events:
{% codeblock fillIn.js %}
function fillIn(app, selector, contextOrText, text) {
  //init code

  run(function() {
    $el.val(text).change();
  });
  return app.testHelpers.wait();
}
{% endcodeblock %}

What I needed was a way for the appropriate key events to be raised after every character.  I might also have code in ```keydown``` and ```keyup``` for example.

With this in mind, I created this test helper to meet my requirements:
{% codeblock fillInWithInputEvents.js %}
function fillInWithInputEvents(app, selector, contextOrText, textOrEvents, events) {
  var $el, context, text;

  if (typeof events === 'undefined') {
    context = null;
    text = contextOrText;
    events = textOrEvents;
  } else {
    context = contextOrText;
    text = textOrEvents;
  }

  if (typeof events === 'string') {
    events = [events];
  }

  $el = app.testHelpers.findWithAssert(selector, context);

  $el.val('');

  focus($el);

  function fillInWithInputEvent(character) {
    var val = $el.val();
    var charCode = character.charCodeAt(0);

    val += character;

    run(function() {
      $el.val(val).change();
    });

    for (var i = 0, l = events.length; i < l; i++) {
      run($el, "trigger", events[i], { keyCode: charCode, which: charCode });
    }
  }

  for (var i = 0, l = text.length; i < l; i++) {
    fillInWithInputEvent(text[i]);
  }

  return app.testHelpers.wait();
}
{% endcodeblock %}

The above helper will input the text one character at a time and raise any supplied events.

For example if I only want to raise an ```input``` event with each character then I would use it like this:
{% codeblock single.js %}
  visit('/')
    .fillInWithInputEvents('input.autosuggest', 'Michael', 'input')
{% endcodeblock %}
Or if I want to raise more than one event then I can supply an array of input events:
{% codeblock multiple.js %}
  visit('/')
    .fillInWithInputEvents('input.autosuggest', 'Michael', ['input', 'keydown'])
{% endcodeblock %}

You can use the helper like <a href="https://github.com/dagda1/emberx-autosuggest/blob/master/tests/test-helper.js" target="_blank">this</a> in an ```ember-cli``` ```test-helper``` and I have created this <a href="https://github.com/emberjs/ember.js/pull/11016" target="_blank">pull request</a> to get it included with the ember source.
