---
layout: post
title: "Higher Kinded Types in typescript"
date: 2018-04-14 14:01:52 +0100
comments: true
categories: typescript javascript
---
Higher kinded types are not currently possible in typescript but let us start by explaining why they exist.

In functional programming there is the concept of a <a href="https://en.wikipedia.org/wiki/Functor" target="_blank">functor</a>.  A functor is simply something that can be mapped over.  In OOP speak, we'd call it a **mappable** or some sort of container such as `Array` that contains a `map` function.

A very simple javascript example would be:

{% codeblock mappable.js %}
const isEven = x => x % 2 === 0

console.log([1,2,3,4].map(isEven))
// => [false, true, false, true]
{% endcodeblock %}

The above example is mapping over an Array and it is mapping from an array of `int` to an array of `bool`.  In this example, the `Array` is the functor.  The functor keeps its shape meaning the same number of elements exist after the mapping operation but each element could potentially change.

A functor can be thought of as a container with a `map` function.  A functor is mapping between categories, meaning that it can map from type `a` to type `b`.  The functor in the above example is the array and the type `a` would be int with type `b` the bool.

In haskell, functor has the following type signature:

{% codeblock functor.hs %}
class Functor f where
  fmap :: (a -> b) -> f a -> f b
{% endcodeblock %}

You can think of `fmap` as **functor map**.

If we consider the type signature for `fmap` again:

{% codeblock fmap.hs %}
fmap :: (a -> b) -> f a -> f b
{% endcodeblock %}

What might not be obvious is that `f`, `a` and `b` from the above are actually type parameters.  Any lowercase type in haskell is a type parameter.

The more specialised `Functor` instance for lists looks like this:

{% codeblock instance.hs %}
instance Functor [] where
  fmap = map
{% endcodeblock %}

The existing list `map` function is used for the `fmap` implementation.  `map` has the following signture:

{% codeblock map.hs %}
map :: (a -> b) -> [a] -> [b]
{% endcodeblock %}

If we now start filling in the type parameters for functor, then the `f` type parameter is the type constructor `[]` or would be the array in the javascript example above.  Remember the functor is the box or container that contains the `fmap` function.

The functor mapping function `(a, b) -> f a -> f b` defines the mapping of a functor of type `a` to a functor of type `b`.  Another way to think of this is an `f` of `a` and `f` of `b` or in this case a list of `a` or a list of `b`.

The more specialised version of `fmap` for list looks like this:

{% codeblock f.hs %}
fmap :: (a -> b) -> [a] -> [b]
{% endcodeblock %}

Hopefully it is clear that `f` is the container or `list` in this specific instance that maps a list of `a` to a list of `b`.

In the above example `f` is the higher kinded type and both `a` and `b` are both type parameters.

A higher kinded type (**HKT**) is simply a type parameter that takes a type parameter.  The equivolence with higher order function **HOF** is that an HOF can take a function as an argument.

## Kinds

In haskell, there are types and there are kinds.  You have concreate types like `Int`, `Bool` and `String`.  All of these are the kind `*`.  Why?  Because they cannot take any type parameters.

`[a]` is a parameterised type because it takes one concrete type and has the form `* -> *` and is called a first order type.

A higher kinded type abstracts the parameterised type `[]` and has the form `(* -> *) -> *`.

In functor, `f` is of kind `(* -> *) -> *` and `a` or `b` are of kind `*` which is often referred to as the ground type.

## Where is the typescript?

Now when we try and define functor in typescript, we quickly come unstuck trying to define an `f` of `a` to an `f` of `b` because typescript does not support higher kinded types or type parameters that take type parameters.

{% codeblock functor.js %}
interface Functor<F> {
  map<A, B>(f: (a: A) => B, fa: ?): ?
}
{% endcodeblock %}

A number of the functional typescript libraries like <a href="https://github.com/gcanti/fp-ts" target="_blank">fp-ts</a> have come up with a similar work around for the lack of higher kinded types.

They all have a similar interface:

{% codeblock HKT.js %}
interface HKT<F, A> {
  readonly _F: F;
  readonly _A: A;
}
{% endcodeblock %}

`HKT` takes 2 type parameters `F` and `A`. Thinking in terms of kinds, `F` represents the first class parameterised type `(* -> *)` and `_A` represents the ground type `*`.

We can now define functor like this:

{% codeblock functor.js %}
export interface Functor<F> {
  map<A, B>(f: (a: A) => B, fa: HKT<F, A>): HKT<F, B>
}
{% endcodeblock %}

The above code is now equivalent to the haskell version, we have a `map` function that maps over a container or box of `A` and returns a container or box of `B`.

Hopefully higher kinded types will land in typescript soon but until then I think this is a very inventive solution to a difficult problem.


