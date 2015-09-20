---
layout: post
title: Functional JavaScript in the browser and why I should care
date: 2015-07-01 21:04:52 +0100
comments: true
categories: JavaScript
---
Why should you care about functional javascript?  I'd heard of the merits of functional programming for a while now and up until recently, I thought that the main benefits of functional programming where really only applicable when dealing with highly concurrent programs when functional programming can help guarantee thread safety.  Writing side effect free functions that use immutable data structures rules out interaction problems between threads that mutate data.  All well and good but why should I care about any of this in the single threaded realm of the browser?

The pendulum between server heavy app and client heavy app. has swung back to the client again for this epoch and single page applications are all the rage.  This would have seemed like insanity at the turn of the century but advances in browser technology and a demand for more ambitious applications has led us to the now infamous ```SPA```.  Our programs live a lot longer in the browser than they used to.  Gone are the days when we could rely on a browser refresh to *reboot* the browser code and rebuild clear all our objects from memory.  ```reactjs``` has come up with a beautfiul top down, one way data push that can simulate this behaviour by abstracting away that gargantuan side effect known as the DOM into a virtual DOM to diff against the previous state.  Reactjs has embraced functional programming and so should you.

## Side effects, function Purity and Referential Transparency
Functional programming favours functions that, for a given input, will consistently return the same output.  A pure function does not depend on and does not modify the state of variables outside of its scope. When a function performs any other action, apart from calculating its return value, the function is impure.  A program or function is said to have side effects if it can produce different outputs given the same input.

A very simple example of a pure function is below:
{% codeblock mult.js %}
const mult = (a, b) => a * b;
{% endcodeblock %}
Referential transparency is the ability to freely replace an expression with its value and not change the behaviour of the program.  We can show that the ```mult``` function is both pure and ```referentially transparent``` with the code below:
{% codeblock rf.js %}
const mult = (a, b) => a * b;

const result = mult(2, 3) + mult(4, 5);

const result1 = 6 + mult(4, 5);

const result2 = 6 + 20;

console.log(result == result1 == result2);
{% endcodeblock %}
Replacing the call to ```mult``` with the result does not alter the output of the computation.

Below is an example of a function that relies on a global variable to calculate its output:
{% codeblock sideeffect.js %}
let number = 0;

const inc = (n) => {
  number = number + n;

  return number;
}

const result1 = inc(1);  //=> 1
const result2 = inc(1);  //=> 2

console.log(result1 === result2);
{% endcodeblock %}
We can clearly see that providing the same input each time does not give us the same output each time.

Side effects are functions that rely on the outside **OR** can be functions that alter the outside world.  The ubiquitous ```Hello World``` program is actually a side effect because it alters the state of the output on the screen.  A call to ```console.log``` is a side effect.  All IO is side effect laden, reading a file from a disk is a side effect because it might throw an exception because the network is unavailable.  We therefore cannot guarantee the result of calling the function.

Any function that does not return a value will more than likely have side effects because it is probably mutating some external state.

Our programs would not be very practical without side effects and the goal of functional programming is not to eliminate side effects but instead we want to limit them and also more importantly, isolate them.  How can we limit side effects in the sea of mutation that is javascript?  The first thing we can do is to use immutable data structrues.

## Immutable Data Structures
I think the penny is starting to drop with many in the javascript world about how sensible immutable data structures are.  A data structure is immutable if it cannot be changed after it has been created.

In the chaotic realm of the single page application, I find it very reassuring to know that after I have created an object or assigned a value to a variable, it will not be changed by some well meaning code outside of my control.  A lot of SPA frameworks use two way data binding to bind objects to DOM elements or other structures.  I have also had hellish results with using observation through ```Object.observe``` or some other mechanism of reacting to state changes, I now consider this to be an anti-pattern.

es6 has introduced the ```const``` keyword which is analogous to ```final``` in java, it does not mean that all data structures defined with the ```const``` keyword are immuable but it does mean they cannot be reassigned once they have been assigned.  The code sample below will illustrate this:
{% codeblock const.js %}
const number = 3;

number = 4; // => error: Attempting to override 'number' which is a constant.

const person = {name: 'Paul Cowan'};

person.name = 'Bob Cowan'; // mutate away

console.log(person.name);

person = {name: 'Paul Cowan'}; //=> error:  "person" is read-only
{% endcodeblock %}
If we want full immutability then we can use something like <a href="https://facebook.github.io/immutable-js/" target="_blank">immutable-js</a> or <a href="https://github.com/swannodette/mori" target="_blank">mori</a>.  Both libraries provide persistent data structures for your use.  The data structures are called persistent because they always presist a previous version of themselves when modified.

Below is an example of ```immutable-js``` that illustrates this:

{% codeblock immutable.js %}
let list1 = Immutable.List.of(1, 2);
let list2 = Immutable.List.of(1, 2);

console.log(Immutable.is(list1, list2) === true);  // => true, equality by value and not reference

let map1 = Immutable.Map({a: 1, b: 2, c: 3});
let map2 = map1.set('b', 2);

console.log(map1 === map2); // => true, map2 returns a reference to map1 because no change was made

let map3 = map1.set('b', 50);

console.log(map1 !== map3);

console.log(map1.toJS()); //=> {a: 1, b: 2, c: 3}, map1 is unchanged
console.log(map3.toJS()); //=> {a: 1, b: 50, c: 3}
{% endcodeblock %}
You can see from the above that we can check equality by value and not reference on ```line4``` when comparing ```list1``` and ```list2```.

Also illustrated is how ```map2``` returns a reference to ```map1``` on ```line 7``` because no change was made to the underlying hash and on ```line 11```, ```map1``` remains unchanged and a new version of the data structure is returned after changing or adding a new value to part of the data strurcture.  I hope you can see how we could use persistent data structures to trivially hookup undo and redo functionality (more on this later).

## Statelessness
Imperative programming works by changing programming state to produce answers.  Conversely, functional programming produces answers through stateless operations.

This brings us back to function purity, our programs are easier to reason about if we can be sure that given the same input, we get the same output.  We want to avoid functions that reference member variables or any thing outside of the functions environment wherever possible.

## Back to the Real world
Here are some things to consider everytime you create a function in javascript:

- Are my functions dependant on the context in which they are called, or are they pure and independent?
- Can I write these functions in such a way that I could depend on them always returning the same result for a given input?
- Am I sure that these functions won't modify anything outside of themselves?
- Pass values as parameters, don't store state in member variables.

I blogged <a href="http://www.thesoftwaresimpleton.com/blog/2015/05/21/functional-ui-ember-1/" target="_blank">here</a> here about using a real world example to limit side effects in an emberjs application and  <a href="http://www.thesoftwaresimpleton.com/blog/2015/03/13/ember-reflux/" target="_blank">here</a> about how you can use persistent data structures in an emberjs application to implement basic undo and redo functionality.

## Higher Order Functions
A higher order function is either:

- A function that takes a function as an argument
- A function that returns a function

I am going to concentrate on the latter and I am going to first of all use <a href="https://en.wikipedia.org/wiki/Partial_application" target="_blank">partial application</a>.

Wikipedia gives the following definition of partial application:
{% blockquote %}
 partial application (or partial function application) refers to the process of fixing a number of arguments to a function, producing another function of smaller arity
{% endblockquote %}

If we return to our ```mult``` function that I showed in the beginning, we can create a new function ```double``` by using partial application:
{% codeblock double.js %}
const mult = (a, b) => a * b;

const double = mult.bind(null, 2);

console.log(double(3)); // => 6
{% endcodeblock %}
On ```line 3``` of the above, I am using the lesser known property of <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind">bind</a> that after the first argument where you can specify a context, every parameter after the first will be prepended to the list of parameters when invoking the bound function.

We could rewrite the ```double``` to return a function like this:
{% codeblock double2.js %}
const mult = (a, b) => a * b;

const makeMultiplier = (a) => {
  return (b) => {
    return mult(a, b);
  }
}

const double = makeMultiplier(2);

console.log(double(3)); // => 6
{% endcodeblock %}
We now have a ```makeMultiplier``` that can create many different types of functions based on ```mult```.

A more practical use of a higher order function is the ```maybe``` function below that can take a function and return a null safe version:

{% codeblock maybe.js %}
const maybe = (fn) => {
  return (input) => {
    if(!input) {
      return;
    }

    return fn.call(this, input);
  };
};

const toLower = i => i.toLowerCase();

console.log(toLower("henry"));

let nullString;

console.log(toLower(nullString)); // TypeError: Cannot read property 'toLowerCase' of undefined

const nullSafeToLower = maybe(toLower);

console.log(nullSafeToLower(nullString)); // returns undefined, no exception
{% endcodeblock %}

Here we create a new null save function that is returned from the ```maybe``` function.  We can now create other functions from this ```maybe``` function that will handle null values.

### Composability
Another benefit of higher order functions is to take 2 or more functions and combine them into one.  Below is an ```and``` function that will perform a logical and on the output of two functions:
{% codeblock and.js %}
const isEven = (number) => number %2 === 0;

const isPositive = (number) => number >= 0;

const and = (f, g) => {
  return function() {
    return f.apply(this, arguments) && g.apply(this, arguments);
  }
};

isPositiveAndEven = and(isPositive, isEven);

console.log(isPositiveAndEven(6)); // => true
console.log(isPositiveAndEven(-4)); // => false
{% endcodeblock %}

### Further down the rabbit Hole
There are a number libraries that go much further with respect to functional programming:

- <a href="http://omniscientjs.github.io/" target="_blank">omniscientjs</a>
    * Built on top of reactjs
    * Top down rendering
    * Immutable Data
- <a href="https://github.com/omcljs/om" trarget="_blank">OM</a>
    * Built on top of reactjs
    * Written clojurescript
    * ClojureScript's immutable data types
- <a href="https://github.com/purescript/purescript" target="_blank">Purescript</a>
    * Inspired by haskell
    * Type system
    * Bindings for reactjs and angular
- <a href="http://elm-lang.org/" target="_blank">Elm</a>
    * Compiles to HTML and javascript
    * Compiler guarantees no runtime exceptions
