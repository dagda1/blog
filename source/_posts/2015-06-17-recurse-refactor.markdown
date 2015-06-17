---
layout: post
title: "Clojure - idiomatic refactor from recursion"
date: 2015-06-17 12:54:40 +0100
comments: true
categories: Clojure
---
I've become addicted to solving programming challenges that you might find at <a href="https://www.hackerrank.com/" target="_blank">hackerrank</a> or <a href="http://www.4clojure.com/" target="_blank">4clojure</a>.  I spend most of my day job in ```emberjs``` and as much as I find myself agreeing with ```reactjs's``` take on things, I simply don't have the energy or the will to learn another javascript framework.  I also think it is bad to learn another way to do the same thing in your spare time.  You need to push your brain into interesting and new avenues.

I'm using ```clojure``` to solve these programs and it is quite nice to not have to worry about persisting data or learning the latest du jour framework.

I have found myself using the same pattern over and over again when iterating a sequence that might decrease due to some condition and accumulating a value while doing this.  Below was my bog standard way of doing this:
{% codeblock recursion.clj %}
(defn process [acc sticks]
  (let [smallest (apply min sticks)
        cuts (filter #(> % 0) (map #(- % smallest) sticks))]

    (if (empty? cuts)
      acc
      (process (conj acc (count cuts)) cuts))))
{% endcodeblock %}
The above code reduces each item in the sequence by the smallest value and then filters out any values that are zero or less (lines 2 and 3).  If there are any values left, the function recursively calls itself until the sequence is empty.  On each call to the function, the count of remaining elements in the sequence is added to an accumulator argument ```acc```.

I have been using this pattern for a while now but it did not seem to me to be idiomatic clojure.  I did some research and the refactoring below is a huge improvement in terms of modularity, readibility and being more idiomatic:
{% codeblock iterate.clj %}
(defn cuts [sticks]
  (let [smallest (apply min sticks)]
    (filter pos? (map #(- % smallest) sticks))))

(defn process [sticks]
  (->> (iterate cuts sticks)
       (map count)         ;; count each sticks
       (take-while pos?))) ;; accumulate while counts are positive
{% endcodeblock %}
The first step was to isolate in a new function the logic that performs the filtering on the sequence.  I have created a ```cuts``` function on ```line 1```.  This improves the modularity.

The ```process``` function on line 5 uses the <a href="https://clojuredocs.org/clojure.core/-%3E%3E" target="_blank">thread last</a> macro.  Both the thread macro ```->``` and the thread last macro ```->>``` make the code easier to read by allowing you to use an imperative style rather than nesting the calls.  The thread last macro inserts the first argument that you pass to it as the last argument to each of the forms.
Below are some simpler examples of the thread last macro that should illustrate how the argument fills in the last argument for each of the forms:
{%codeblock last.clj %}
(->> x
     f)
;; (f x)
(->> x
     f
     g)
;; (g (f x))
(->> x
     (f y))
;; (f y x)
{% endcodeblock %}

If we return to our example, we can use <a href="https://clojuredocs.org/clojure.core/macroexpand" target="_blank">```macroexpand```</a> to expand out the macro to its nested reality:
{%codeblock macro.clj%}
(macroexpand '(->>
              (iterate cuts [1 2 3 4 3 3 2 1])
              (map count)
              (take-while pos?)))
{%endcodeblock%}
Below is the output of the ```macroexpand``` call:
{% codeblock result.clj %}
(take-while pos? (map count (iterate cuts [1 2 3 4 3 3 2 1])))
{% endcodeblock %}

The first argument in this case is supplied via the <a href="https://clojuredocs.org/cl
ojure.core/iterate" target="_blank">iterate</a> function on ```line 6``` that generates an infinite lazy sequence by taking two parameters, a function and a seed value.  The seed value in this case is the vector ```[1 2 3 4 3 3 2 1]``` and this is initially passed to the ```cuts``` function. ```iterate``` will then infinitely call ```cuts``` passing in as an argument, the previous call to ```cuts```.

```map``` is used to apply a function to each item returned from the ```iterate``` form and we use ```count``` as the function to transform the result of each call to ```cuts```.  ```take-while``` is used to constrain the infinite nature of ```iterate``` and as soon as the ```pos?``` function returns false, the results are returned to the calling function.  We use ```pos?``` to check whether there are are any elements that are not zero.  ```take-while``` will also accumulate the ```count``` results and return them to the calling function instead of using an accumulator argument like I used in my original example.

I really like how this turned out and it is definitely more idiomatic clojure than my original version and I think this is a good barrier to get past in my journey into clojure.
