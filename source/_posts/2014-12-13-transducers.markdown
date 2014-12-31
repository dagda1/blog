---
layout: post
title: "Clojurescript - Using Transducers To Transform Native Javascript Arrays"
date: 2014-12-13 20:56:40 +0000
comments: true
categories: clojure clojurescript
---
I've been digging into clojurescript more and more but as I am still quite new to cljs, I find myself over zealously calling the <a href="https://github.com/clojure/clojurescript/blob/master/src/cljs/cljs/core.cljs#L8515">clj->js</a> and <a href="https://github.com/clojure/clojurescript/blob/master/src/cljs/cljs/core.cljs#L8539" target="_blank">js->clj</a> interop functions that transform javascript arrays to clojurescript vectors, maps, lists etc. and vice versa.  I found this frustrating as I want to use the power of the clojurescript language and not have to drill down to javascript unless it absolutely necessary.

I've been writing a <a href="https://github.com/facebook/react">react</a> wrapper component which has pretty much dictated that I need to be dealing with native javascript objects at all times and as such I am having to call these interop functions.  An example of this is in the code below:
{% codeblock keys.cljs %}
(let [prevKeys (.keys js/Object (or prevChildMapping (js-obj)))
      nextKeys (.keys js/Object (or nextChildMapping (js-obj)))
      keysToEnter (clj->js (filter #(not (.hasOwnProperty prevChildMapping %)) nextKeys))
      keysToLeave (clj->js (filter #(not (.hasOwnProperty nextChildMapping %)) prevKeys))]

      (set! (.-keysToEnter this) keysToEnter)
      (set! (.-keysToLeave this) keysToLeave)))))
{% endcodeblock %}
On lines 3 and 4, I am calling <a href="https://github.com/clojure/clojurescript/blob/master/src/cljs/cljs/core.cljs#L8515">clj->js</a> to transform a clojurescript ```PersistentVector``` into the javascript native array equivalent.  What I really wanted was to call the clojurescript sequence functions ```map```, ```reduce```, ```filter``` etc. on native javascript objects.  I asked if this was possible in the clojurescript irc and <a href="http://clojure.org/transducers" target="_blank">transducrs</a> were put forward as a means of achieving the goal.

###Transducers
I had heard of transducers in the clojure world without taking the trouble to see what all the fuss was about but I had no idea that they were available in clojurescript.  I'm now going to give a brief introduction as to what transducers are but there is lots of good material out there that probably do a better job and Rich Hickey's <a href="https://www.youtube.com/watch?v=6mTbuzafcII">strangeloop</a> introduction to them is a great start.

I always address a new concept by first of all determining what problem does the new concept solve and with tranducers the problem is one of decoupling.  You are probably familiar with ```filter``` which returns all items in a collection that are true in terms of a predicate function:
{% codeblock filter.cljs %}
(filter odd? (range 0 10)) ;=> (1 3 5 7 9)
{% endcodeblock %}
It should be noted that ```filter``` could be constructed using ```reduce```.
{% codeblock filter-odd.cljs %}
(defn filter-odd
  [result input]
  (if (odd? input)
    (conj result input)
    result))

(reduce filter-odd [] (range 0 10))
{% endcodeblock %}
The problem with the above is that we cannot replace ```conj``` on line 4 with another builder function like```+```.  This problem holds true for all the pre-transducer sequence functions like ```map```, ```filter``` etc.  Transducers set out to abstract away operations like ```conj``` so that the creation of the resultant datastructure is decoupled from the ```map```/```filter``` logic.

```conj``` and ```+``` are reducing functions in that they take a result and an input and return a new result.  We could refactor our ```filter-odd``` function to a more generic ```filtering``` function that allows us to supply different predicates and reducing funtions by using higher order functions:
{% codeblock filtering.cljs %}
(defn filtering
  [predicate]
  (fn [reducing]
    (fn [result input]
      (if (predicate input)
        (reducing result input)
        result))))

(reduce ((filtering odd?) conj) [] (range 0 10)) ;=>[1 3 5 7 9]
(reduce ((filtering even?) +) 0 (range 0 10)) ; => 20
{% endcodeblock %}
The above is not as scary as it looks and you can see on lines 9 and 10 that we are able to supply different reducing functions (```conj``` and ```+```).  This is the problem that transducers set out to solve, the reducing function is now abstracted away so that the creation of the datastructure is decoupled from the sequence function (```filter```, ```map``` etc.) logic.

As of clojure 1.7.0 most of the core sequence functions (```map```, ```filter``` etc.) are gaining a new 1 argument arity that will return a transducer that, for example this call will return a transducer from ```filter```:
{% codeblock odd.cljs %}
(filter odd?)
{% endcodeblock %}

One of the new  ways (but not the only way) to apply transducers is with the <a href="http://clojure.github.io/clojure/branch-master/clojure.core-api.html#clojure.core/transduce" target="_blank">transduce</a> function.  The transduce function takes the following form:
{% codeblock transduce.cljs %}
transduce(xform, f, init, coll)
{% endcodeblock %}
The above states that ```transduce``` will reduce a collection ```coll``` with the inital value ```init```, applying a transformation ```xform``` to each value and applying the reducing function ```f```.

We can now apply this to our previous example

{% codeblock xform.cljs %}
(def xform
  (filter odd?))

(transduce xform + 0 (range 0 10)) ;=> 25
(transduce xform conj [] (range 0 10)) ;=>  ;=>[1 3 5 7 9]
{% endcodeblock %}
I hope it is obvious that ```(range 0 10)``` is ```coll``` and ```[]``` is the ```init```, ```xform``` is the transducer function and ```+``` or ```conj``` are the reducing functions.

###Meanwhile Back in Javascript land......
If we now shift back to our specific example, we can use a transducer to transform a native javascript array because a transducer is fully decoupled from input and output sources.

This is the current code that we want to refactor:
{% codeblock cljs.cljs %}
(clj->js (filter #(not (.hasOwnProperty prevChildMapping %)) nextKeys))
{% endcodeblock %}

So the first question is what would the reducing function be when dealing with native arrays?  The answer is the native array ```push``` push method which adds a new item to an array.  My first ill thought out attempt at the above looked something like this:
{% codeblock transduce.cljs %}
(transduce (filter #(not (.hasOwnProperty prevChildMapping %))) (.-push #js[]) #js [] nextKeys)
{% endcodeblock %}

This is completely wrong because I had not grasped what is required of the reducing funcion.  A reducing function takes a result and an input and returns a new result e.g.
{% codeblock conj.cljs %}
(conj [1 2 3] 4) ;=> [1 2 3 4]
(+ 10 1) ;=> 11
{% endcodeblock %}
The ```push``` function does not satisfy what is required as the ```push``` function actually returns the length of the array which is not what is expected.  What was needed was someway of turning the ```push``` function into a function that behaved in a way that the transducer expected.  The push function would need to return the result:
{% codeblock arr.cljs %}
(fn [arr x] (.push arr x) arr)
{% endcodeblock %}

But as it turns out, this also does not work because a reducing function to transduce has to have 0, 1 and 2 arities and our reducing function only has 1.

As it turns out, both clojure and clojurescript provide a function called <a href="https://clojure.github.io/clojure/branch-master/clojure.core-api.html#clojure.core/completing" target="_blank">completing</a> that takes a function and returns a function that is suitable for transducing by wrapping the reducing funtion and adding an extra arity that simply calls the <a href="https://clojuredocs.org/clojure.core/identity" target="_blank">identity</a> function behind the scenes.  Below is the ```completing``` function from the ```clojure.core``` source.
{% codeblock completing.cljs %}
(defn completing
  ([f] (completing f identity))
  ([f cf]
     (fn
       ([] (f))
       ([x] (cf x))
       ([x y] (f x y)))))
{% endcodeblock %}
My final code ended up looking like this:
{% codeblock keys.cljs %}
(keysToEnter (transduce (filter #(not (.hasOwnProperty prevChildMapping %))) (completing (fn [arr x] (.push arr x) arr)) #js [] nextKeys)
{% endcodeblock %}

The reducing function that uses the native javascript push function is wrapped in ```completing``` that makes it suitable for transducing.

I think I've ended up with more code than I started with and I also think that this is a poor example of transducers but I wanted to outline the mechanics involved in using transducers with native javascript arrays as I could find absolutely nothing on the google etc. so hopefully this will point somebody else in the right direction.

If I have got anyting wrong in this explanation then please leave a comment below.