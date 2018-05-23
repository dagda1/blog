---
layout: post
title: "TypeProps - Better Higher Kinded Types for typescript"
date: 2018-05-10 06:15:48 +0100
comments: true
categories: typescript javascript
---

In my [last post](http://www.thesoftwaresimpleton.com/blog/2018/04/14/higher-kinded-types/) I described the solution put forward by <a href="https://github.com/gcanti/fp-ts" target="_blank">fp-ts</a> for higher kinded types in typescript.

<!-- TODO: EXPLAIN WHY IT IS BAD-->

I've since then come across <a href="https://github.com/SimonMeskens/TypeProps" target="_blank">TypePrpos</a> which is really pushing the boundaries of what is currently possible in typescript.

<!-- TODO: TypeProps intro -->

## TypeProps type dictionary

The base abstraction in **TypeProps** is the `TypeProp` type dictionary that is listed below:

{% codeblock TypeProps.js %}
// Plumbing
interface TypeProps<T = {}, Params extends ArrayLike<any> = never> {
    array: {
        infer: T extends Array<infer A> ? [A] : never;
        construct: Params[0][];
    };
    null: {
        infer: null extends T ? [never] : never;
        construct: null;
    };
    undefined: {
        infer: undefined extends T ? [never] : never;
        construct: undefined;
    };
    unfound: {
        infer: [NonNullable<T>];
        construct: Params[0];
    };
}
{% endcodeblock %}

TypeScript takes 2 gereric arguments, the first is the ubiquitous `T` argument.  

The assingnment `T = {}` is worth explaining if you are unfamiliar with it.  In typescript generic arguments can take default values so if no type is provided for `T` then it is assigned the type `{}` which describes an object with no properties.  

The type dictionary contains a property for the basic types like `array` and `null`.  Each `type` has two properties, `infer` and `construct`:

