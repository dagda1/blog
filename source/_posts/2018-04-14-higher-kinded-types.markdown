---
layout: post
title: "Higher Kinded Types in typescript""
date: 2018-04-14 14:01:52 +0100
comments: true
categories: typescript javascript
---
Higher kinded types are not currently possible in typescript but first of all let's explain why they exist.

## The Problem

In functional programming there is concept of a <a href="https://en.wikipedia.org/wiki/Functor" target="_blank">functor</a>.  A functor is simply something that can be mapped over.  In OOP speak, we'd call it a **mappable**.

A very simple javascript example would be:

{% codeblock mappable.js %}
console.log([ 2, 4, 6 ].map(x => x + 3))
// => [ 5, 7, 9 ]
{% endcodeblock %}

The above example is mapping over an Array so you would be right in saying that an Array is a functor.  But the same could be said for an object or even functions so a functor can be thought of as a container with a `map` function.

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

The existing `map` function is used.  `map` has the following signture:

{% codeblock map.hs %}
map :: (a -> b) -> [a] -> [b]
{% endcodeblock %}

If we now start filling in the type parameters for functor, then the `f` type parameter is the type constructor `[]` or list or you could think of it as an array.  Remember the functor is the box or container that contains the `fmap` function.

We also see `f a -> f b`, which you can think of as an `f` of `a` and an `f` of `b` or in this case a list of `a` or a list of `b`.

The more specialised version of `fmap` for list looks like this:

{% codeblock f.hs %}
fmap :: (a -> b) -> [a] -> [b]
{% endcodeblock %}

Hopefully this is clear that `f` is the container or `list` in this instance that maps a list of `a` to a list of `b`.

In the above example `f a` and `f b`, `f` is the higher kinded type and both `a` and `b` are type parameters.

A higher kinded type **HKT** is simply type parameter that takes a type parameter.  The equivolence with higher order function **HOF** is that an HOF takes a function as an argument.

## Kinds

In haskell, there are types and there are kinds.  You have concreate types like `Int`, `Bool`, `String`.  All of these are the kind `*`.  Why?  Because they cannot take any type parameters.

`[a]` is a parameterised type because it takes one concrete type and has the form `* -> *` and is called a first order type.

A higher kinded type abstracts the parameterised type `[]` and the form `(* -> *) -> *`.`.

In functor, `f` is of kind `(* -> *) -> *` and `a` or `b` are of kind `*`.

## Where is the tyepscript?

Now if we were to try and define functor in typescript, we quickly come unstuck trying to define `f a` because typescript does not support higher kinded types or type parameters that take type parameters.

{% codeblock functor.js %}
interface Functor<F> {
  map<A, B>(f: (a: A) => B, fa: ?): ?
}
{% endcodeblock %}

A number of the functional libraries like <a href="https://github.com/gcanti/fp-ts" target="_blank">fp-ts</a> have come up with a similar work around for the lack of higher kinded types.

They all have a similar interface:

{% codeblock HKT.js %}
interface HKT<F, A> {
  readonly _F: F;
  readonly _A: A;
}
{% endcodeblock %}

`HKT` takes 2 type parameters `F` an `A`. Thinking in terms of kinds, `F` represents the first class parameterised type `(* -> *)` and `_A` represents the ground type `*`.

We can now define functor like this:

{% codeblock functor.js %}
export interface Functor<F> {
  map<A, B>(f: (a: A) => B, fa: HKT<F, A>): HKT<F, B>
}
{% endcodeblock %}

The above code is now equivalent to the haskell version, we have a `map` function that maps over a container or box of `A` and returns a container or box of `B`.



